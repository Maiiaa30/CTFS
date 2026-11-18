# TryHackMe: ROOM_NAME — Writeup

**Room:** [ROOM_NAME](https://tryhackme.com/room/ROOM_CODE)
**Difficulty:** Easy / Medium / Hard
**Category:** e.g. Web Exploitation, Linux Privilege Escalation

## Overview

2–4 sentences: what the box is, the overall attack path at a high level, and the
main vulnerabilities chained. A recruiter should understand the whole story from
this paragraph alone.

## Skills Demonstrated

- Bullet each concrete skill/technique you used
- e.g. Service enumeration with Nmap
- e.g. SQL injection → authentication bypass
- e.g. Linux privilege escalation via sudo misconfiguration

---

## 1. Recon & Enumeration

What you scanned and found. Show the commands and the *relevant* output only.

```bash
nmap -sC -sV -oN nmap/initial TARGET_IP
```

Explain what the results told you and what you decided to pursue next.

## 2. Foothold / Initial Access

How you got the first shell or access. Show the exploit, the payload, the request.

```bash
# the actual command / request used
```

**Lesson:** one line on the real-world root cause and how to prevent it.

## 3. Privilege Escalation

How you went from low-priv user to root/SYSTEM.

```bash
# enumeration that revealed the vector (LinPEAS output, sudo -l, etc.)
# the exploitation step
```

**Lesson:** why this misconfiguration is dangerous in production.

---

## Summary of Vulnerabilities

| # | Vulnerability | Impact |
|---|---|---|
| 1 | e.g. Weak credentials on SSH | Initial access |
| 2 | e.g. SUID binary misconfiguration | Privilege escalation to root |

## Flags

- User flag: `THM{redacted}`
- Root flag: `THM{redacted}`

_Flags redacted per CTF writeup etiquette. Complete the room yourself to retrieve them._

---

<!--
TIPS for a strong writeup (delete before publishing):
- Show your reasoning, not just commands. "I saw X, so I tried Y" is what proves skill.
- Keep output snippets short — only the lines that mattered.
- Every vuln gets a one-line real-world "Lesson" — this is what separates a
  learner from a pentester in a recruiter's eyes.
- Redact flags. Never paste full flag values.
- Screenshots: put images in an ./img/ subfolder and reference them.
-->
