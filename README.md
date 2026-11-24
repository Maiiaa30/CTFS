<!-- Repo suggestion: name it `tryhackme-writeups` or `ctf-writeups` on github.com/Maiiaa30 -->

# TryHackMe Writeups & CTF Portfolio

### Diogo Rodrigues — Junior Penetration Tester / Red Teamer · Braga, Portugal

<p>
  <img alt="Rooms completed" src="https://img.shields.io/badge/TryHackMe_rooms-140%2B-c11b17">
  <img alt="Focus" src="https://img.shields.io/badge/focus-Web_·_PrivEsc_·_AI%2FLLM-1f6feb">
  <img alt="Writeups" src="https://img.shields.io/badge/writeups-4_published-2ea043">
  <img alt="PenTest+" src="https://img.shields.io/badge/CompTIA_PenTest%2B-in_progress-f5a623">
</p>

A curated portfolio of hands-on offensive-security work across **140+ TryHackMe
rooms** — covering web application exploitation, Linux and Windows privilege
escalation, Active Directory, network attacks, and a dedicated focus on the
emerging field of **AI / LLM security**.

Every published writeup documents the full engagement in my own words and
methodology — **recon → enumeration → exploitation → privilege escalation →
remediation lessons** — written to a standard a hiring team can read like a
condensed pentest report. Each vulnerability is paired with a one-line real-world
*Lesson* on root cause and defence. Flags and credentials are redacted per CTF
etiquette.

**Contact**
&nbsp;·&nbsp; 🔗 [TryHackMe](https://tryhackme.com/p/Maiaa)
&nbsp;·&nbsp; 🔗 [LinkedIn](https://www.linkedin.com/in/diogorodrigues-6a3964412)
&nbsp;·&nbsp; 💻 [GitHub](https://github.com/Maiiaa30)
&nbsp;·&nbsp; 📧 diogomaia2003@gmail.com

---

## 📊 At a glance

| | |
|---|---|
| **Rooms completed** | 140+ on TryHackMe ([profile](https://tryhackme.com/p/Maiaa)) |
| **Primary focus** | Web app exploitation · Linux/Windows privesc · AI/LLM security |
| **Methodology** | Recon → enumeration → exploitation → privilege escalation → remediation |
| **Core tools** | Burp Suite · Nmap · Metasploit · Gobuster/ffuf · Hydra · Hashcat · Wireshark · SQLMap · LinPEAS/WinPEAS |
| **Scripting** | Python · Bash |
| **Certification** | CompTIA PenTest+ (in progress) · Rumos Penetration Testing course |

### Skills demonstrated across rooms
`Web App Pentesting` · `SQL Injection` · `XSS` · `SSRF` · `IDOR` · `CSRF` · `LFI/RFI → RCE` · `JWT attacks` · `Authentication bypass` · `Linux Privilege Escalation` · `Windows Privilege Escalation` · `Active Directory` · `Network Enumeration` · `Password Cracking` · `Prompt Injection` · `LLM Jailbreaking` · `AI Supply-Chain Security` · `OWASP Top 10`

---

## ⭐ Featured writeups

### [Domino](https://tryhackme.com/room/domino) — Medium — Web + Linux PrivEsc
Full kill-chain: hardcoded key in client JS → AES-ECB config decryption → **JWT signature-bypass** (role forgery) → **RFI → RCE** → admin-bot session hijack → credential reuse → group-writable root cron = **full root compromise**.
📄 **[Read the writeup »](./Domino/Readme.md)**

### [The Marketplace](https://tryhackme.com/room/marketplace) — Medium — Web + PrivEsc + Docker Escape
**Stored XSS** → steal admin cookie via the report feature → **SQL injection** leaks an SSH password hidden in private messages → credential reuse → **`tar` wildcard injection** (sudo) → **Docker group escape** to root.
📄 **[Read the writeup »](./Marketplace/README.md)**

### [Watcher](https://tryhackme.com/room/watcher) — Medium — Web + Linux PrivEsc
**LFI** in `?post=` reads a protected file leaking **FTP creds** → upload a PHP shell to a writable FTP dir and trigger it via **path-traversal include** (LFI → RCE) → **four privesc hops**: writable cron script → **Python module hijack** of a sudo script → root **SSH key** in `/opt/backups`.
📄 **[Read the writeup »](./Watcher/README.md)**

### [Empline](https://tryhackme.com/room/empline) — Medium — Web + Linux PrivEsc
**Virtual-host enumeration** uncovers a hidden `job.` subdomain running **OpenCATS** → known **RCE** (Exploit-DB 50585) for a foothold → privesc through a rare Linux **file capability**: `cap_chown` on the `ruby` binary is abused to seize `/etc/shadow`, rewrite the root hash, and `su` to **root**.
📄 **[Read the writeup »](./Empline/README.md)**

---

## 📚 Full index

> ✅ = writeup published · 📝 = completed, writeup planned. Every room links to TryHackMe.
> This index mirrors my completed rooms. Full writeups are published progressively, prioritising the rooms that best demonstrate methodology — **4 published so far** ([Domino](./Domino/Readme.md), [Marketplace](./Marketplace/README.md), [Watcher](./Watcher/README.md), [Empline](./Empline/README.md)).

<details open>
<summary><h3>🤖 AI / LLM Security (30+ rooms)</h3></summary>

A fast-growing, under-covered field. Prompt injection, jailbreaking, RAG/supply-chain attacks, and AI threat modelling — red, blue and purple.

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [BankGPT](https://tryhackme.com/room/bankgpt) | Easy | LLM app abuse | 📝 |
| [Evil-GPT v2](https://tryhackme.com/room/hfb1evilgptv2) | Easy | Prompt injection | 📝 |
| [Oracle 9](https://tryhackme.com/room/oracle9) | Easy | AI pentesting | 📝 |
| [White Rabbit](https://tryhackme.com/room/whiterabbit) | Medium | Prompt injection | 📝 |
| [LLMborghini](https://tryhackme.com/room/llmborghini) | Easy | Indirect prompt injection | 📝 |
| [CupidBot](https://tryhackme.com/room/lafb2026e6) | Easy | LLM abuse | 📝 |
| [The Concierge Knows Too Much](https://tryhackme.com/room/hh-theconciergeknows-2d7eb4d9) | Easy | AI pentesting | 📝 |
| [UnIndexed](https://tryhackme.com/room/unindexedchallenge) | Medium | AI data exposure | 📝 |
| [Lockdown](https://tryhackme.com/room/lockdownai) | Medium | AI defence | 📝 |
| [Prompt Injection](https://tryhackme.com/room/promptinjectionls) | Medium | Prompt injection | 📝 |
| [Jailbreaking](https://tryhackme.com/room/jailbreaking) | Medium | Safety bypass | 📝 |
| [Prompt Engineering](https://tryhackme.com/room/promptengineeringaisec) | Easy | Adversarial prompting | 📝 |
| [Prompt Defence](https://tryhackme.com/room/promptdefence) | Medium | Blue-team defence | 📝 |
| [LLM Security](https://tryhackme.com/room/llmsecurity) | Medium | LLM attack surface | 📝 |
| [Securing AI Systems](https://tryhackme.com/room/securingaisystems) | Medium | OWASP/ATLAS, secure design | 📝 |
| [AI Security Threats](https://tryhackme.com/room/aisecuritythreats) | Easy | Threat landscape | 📝 |
| [AI Models & Data](https://tryhackme.com/room/aimodelsdata) | Medium | Data/model security | 📝 |
| [The Building Blocks of AI](https://tryhackme.com/room/aimlsecuritythreats) | Easy | AI/ML/LLM fundamentals | 📝 |
| [AI Forensics](https://tryhackme.com/room/aiforensics) | Medium | AI DFIR | 📝 |
| [AI System Reconnaissance](https://tryhackme.com/room/aisystemreconnaissance) | Medium | ML service recon | 📝 |
| [AI Threat Modelling](https://tryhackme.com/room/aithreatmodelling) | Medium | MITRE ATLAS | 📝 |
| [AI Threat Modelling Assessment](https://tryhackme.com/room/aithreatmodellingassessment) | Easy | Applied threat modelling | 📝 |
| [RAG Security Fundamentals](https://tryhackme.com/room/ragsecurityfundamentals) | Easy | RAG attack surface | 📝 |
| [Data Poisoning in RAG Systems](https://tryhackme.com/room/datapoisoninginragsystems) | Medium | Embedding poisoning | 📝 |
| [Sensitive Information Disclosure](https://tryhackme.com/room/sensitiveinformationdisclosure) | Medium | RAG data leakage | 📝 |
| [Understanding AI Supply Chains](https://tryhackme.com/room/understanding-ai-supplychains) | Easy | Supply-chain risk | 📝 |
| [Supply Chain Attack Vectors](https://tryhackme.com/room/supplychain-attack-vectors) | Medium | ML component abuse | 📝 |
| [Securing the AI Supply Chain](https://tryhackme.com/room/securing-the-ai-supplychain) | Medium | Model/dependency audit | 📝 |
| [Payload](https://tryhackme.com/room/payload) | Easy | AI supply-chain breach | 📝 |
| [Checkpoint](https://tryhackme.com/room/checkpoint) | Easy | AI risk triage | 📝 |

</details>

<details>
<summary><h3>🌐 Web Application Exploitation (boxes & challenges)</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Domino](https://tryhackme.com/room/domino) | Medium | JWT bypass · RFI→RCE · chain | ✅ |
| [The Marketplace](https://tryhackme.com/room/marketplace) | Medium | Stored XSS · SQLi · tar wildcard · Docker escape | ✅ |
| [Watcher](https://tryhackme.com/room/watcher) | Medium | LFI→RCE · path traversal · cron · Python hijack · SSH key | ✅ |
| [Empline](https://tryhackme.com/room/empline) | Medium | Vhost enum · OpenCATS RCE · cap_chown privesc | ✅ |
| [Pickle Rick](https://tryhackme.com/room/picklerick) | Easy | Web enum · command exec | 📝 |
| [NahamStore](https://tryhackme.com/room/nahamstore) | Medium | Bug-bounty web | 📝 |
| [Injectics](https://tryhackme.com/room/injectics) | Medium | SQLi · SQLMap | 📝 |
| [Light](https://tryhackme.com/room/lightroom) | Easy | SQL injection | 📝 |
| [El Bandito](https://tryhackme.com/room/elbandito) | Hard | Web exploitation | 📝 |
| [Surfer](https://tryhackme.com/room/surfer) | Easy | SSRF · ffuf · code review | 📝 |
| [MD2PDF](https://tryhackme.com/room/md2pdf) | Easy | SSRF | 📝 |
| [CyberLens](https://tryhackme.com/room/cyberlensp6) | Easy | Web exploitation | 📝 |
| [Agent T](https://tryhackme.com/room/agentt) | Easy | Web misconfig | 📝 |
| [Epoch](https://tryhackme.com/room/epoch) | Easy | Command injection | 📝 |
| [Flip](https://tryhackme.com/room/flip) | Easy | Exploitation · code review | 📝 |
| [Recruit](https://tryhackme.com/room/recruitwebchallenge) | Medium | Web recon → access | 📝 |
| [U.A. High School](https://tryhackme.com/room/yueiua) | Easy | Web exploitation | 📝 |
| [Corridor](https://tryhackme.com/room/corridor) | Easy | IDOR / JWT | 📝 |
| [Lo-Fi](https://tryhackme.com/room/lofi) | Easy | LFI | 📝 |
| [The Sticker Shop](https://tryhackme.com/room/thestickershop) | Easy | Blind XSS | 📝 |
| [CyberHeroes](https://tryhackme.com/room/cyberheroes) | Easy | Auth bypass | 📝 |
| [Checkmate](https://tryhackme.com/room/checkmate) | Easy | Web · Hashcat · CeWL | 📝 |
| [Fools Mate](https://tryhackme.com/room/foolsmate) | Easy | Web enum | 📝 |
| [Fools Mate, Revenge](https://tryhackme.com/room/foolsm8v2) | Medium | Web · Burp · exploitation | 📝 |
| [Room 404](https://tryhackme.com/room/hh-room404-804573bf) | Easy | Web app security | 📝 |
| [Complimentary](https://tryhackme.com/room/hh-complimentary-05e0b604) | Easy | AWS cloud pentest | 📝 |
| [TryHeartMe](https://tryhackme.com/room/lafb2026e5) | Easy | Web challenge | 📝 |
| [Hidden Deep Into my Heart](https://tryhackme.com/room/lafb2026e9) | Easy | Web challenge | 📝 |
| [Ignite](https://tryhackme.com/room/ignite) | Easy | Web → shell | 📝 |
| [Startup](https://tryhackme.com/room/startup) | Easy | Web → foothold | 📝 |
| [battery](https://tryhackme.com/room/battery) | Medium | Web exploitation | 📝 |
| [Web Server Attacks - I](https://tryhackme.com/room/webserverattacks) | Medium | Apache/Nginx/Node misconfig | 📝 |
| [Web Server Attacks - II](https://tryhackme.com/room/webserverattacks2) | Medium | IIS · WebDAV shell | 📝 |
| [Modern Web Stacks](https://tryhackme.com/room/modernwebstacks) | Easy | MERN/Next/Django/LAMP CVEs | 📝 |
| [Guided Pentest: Web](https://tryhackme.com/room/guidedpentestweb) | Easy | Recon → full compromise | 📝 |

</details>

<details>
<summary><h3>🧰 Web Vulnerability Techniques (focused labs)</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [SQL Injection Introduction](https://tryhackme.com/room/sqlinjectionintroduction) | Easy | SQLi · auth bypass · exfil | 📝 |
| [XSS Introduction](https://tryhackme.com/room/xssintroduction) | Medium | Cross-site scripting | 📝 |
| [Intro to SSRF](https://tryhackme.com/room/ssrfqi) | Easy | Server-side request forgery | 📝 |
| [IDOR](https://tryhackme.com/room/idor) | Easy | Insecure direct object ref | 📝 |
| [CSRF Introduction](https://tryhackme.com/room/csrfintroduction) | Easy | Cross-site request forgery | 📝 |
| [Content Discovery](https://tryhackme.com/room/contentdiscoveryx) | Easy | dirb/Gobuster/ffuf/OSINT | 📝 |
| [Walking An Application](https://tryhackme.com/room/walkinganapp) | Easy | Manual browser review | 📝 |
| [Basic Vulnerability Identification](https://tryhackme.com/room/basicvulnerabilityidentificationtechniques) | Easy | Vuln ID with Burp | 📝 |
| [OWASP Juice Shop](https://tryhackme.com/room/owaspjuiceshop) | Easy | Classic web vuln lab | 📝 |

</details>

<details>
<summary><h3>🐧 Linux Boxes & Privilege Escalation</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Mr Robot CTF](https://tryhackme.com/room/mrrobot) | Medium | Web → Metasploit → root | 📝 |
| [Basic Pentesting](https://tryhackme.com/room/basicpentestingjt) | Easy | Enum · Hydra · privesc | 📝 |
| [Simple CTF](https://tryhackme.com/room/easyctf) | Easy | Web · Linux privesc | 📝 |
| [Brooklyn Nine Nine](https://tryhackme.com/room/brooklynninenine) | Easy | Linux box | 📝 |
| [Bounty Hacker](https://tryhackme.com/room/cowboyhacker) | Easy | Hydra · Nmap · privesc | 📝 |
| [Agent Sudo](https://tryhackme.com/room/agentsudoctf) | Easy | Enum · privesc | 📝 |
| [Chill Hack](https://tryhackme.com/room/chillhack) | Easy | Command filter bypass | 📝 |
| [Chocolate Factory](https://tryhackme.com/room/chocolatefactory) | Easy | Hashcat · Nmap · privesc | 📝 |
| [Cyborg](https://tryhackme.com/room/cyborgt8) | Easy | Encrypted archive · source | 📝 |
| [Lian_Yu](https://tryhackme.com/room/lianyu) | Easy | ffuf · forensics | 📝 |
| [GamingServer](https://tryhackme.com/room/gamingserver) | Easy | ffuf · John · privesc | 📝 |
| [Boiler CTF](https://tryhackme.com/room/boilerctf2) | Medium | Web · Linux box | 📝 |
| [Whiterose](https://tryhackme.com/room/whiterose) | Easy | Linux box | 📝 |
| [Hijack](https://tryhackme.com/room/hijack) | Easy | Misconfig · identity | 📝 |
| [Cat Pictures 2](https://tryhackme.com/room/catpictures2) | Easy | Web · Linux | 📝 |
| [mKingdom](https://tryhackme.com/room/mkingdom) | Easy | Reverse shell · privesc | 📝 |
| [Wonderland](https://tryhackme.com/room/wonderland) | Medium | Red-team box | 📝 |
| [Relevant](https://tryhackme.com/room/relevant) | Medium | Pentest challenge | 📝 |
| [Beach Bar](https://tryhackme.com/room/hh-beachbar-d849f7f7) | Easy | Nmap · privesc · enum | 📝 |
| [Linux PrivEsc](https://tryhackme.com/room/linprivesc) | Medium | 8+ privesc techniques | 📝 |
| [Linux PrivEsc: Basics](https://tryhackme.com/room/linprivbasics) | Medium | Enum → exploitation | 📝 |
| [Linux PrivEsc: Enumeration](https://tryhackme.com/room/linprivenum) | Easy | Host enumeration | 📝 |
| [Guided Pentest: Infrastructure](https://tryhackme.com/room/guidedpentestinfrastructure) | Easy | Nmap · Metasploit | 📝 |

</details>

<details>
<summary><h3>🪟 Windows & Active Directory</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Active Directory Basics](https://tryhackme.com/room/winadbasics) | Easy | AD fundamentals | 📝 |
| [Proxy](https://tryhackme.com/room/proxychallenge) | Easy | AD · service account → DC | 📝 |
| [Windows Privilege Escalation](https://tryhackme.com/room/windowsprivesc20) | Medium | WinPEAS · privesc | 📝 |
| [Windows Local Persistence](https://tryhackme.com/room/windowslocalpersistence) | Medium | Persistence techniques | 📝 |
| [Year of the Owl](https://tryhackme.com/room/yearoftheowl) | Hard | Windows box | 📝 |

</details>

<details>
<summary><h3>🔍 Recon, Scanning & Network</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Passive Reconnaissance](https://tryhackme.com/room/passiverecon) | Easy | whois · dig · OSINT | 📝 |
| [Active Reconnaissance](https://tryhackme.com/room/activerecon) | Easy | traceroute · ping · telnet | 📝 |
| [Nmap Live Host Discovery](https://tryhackme.com/room/nmap01) | Medium | ARP/ICMP/TCP discovery | 📝 |
| [Nmap Basic Port Scans](https://tryhackme.com/room/nmap02) | Easy | TCP/UDP scans | 📝 |
| [Nmap Advanced Port Scans](https://tryhackme.com/room/nmap03) | Medium | Null/FIN/Xmas/idle | 📝 |
| [Nmap Post Port Scans](https://tryhackme.com/room/nmap04) | Medium | Service/OS detect · NSE | 📝 |
| [Vulnerability Scanning Tools](https://tryhackme.com/room/vulnerabilityscanningtools) | Medium | Nmap · OpenVAS · Nikto | 📝 |
| [Protocols and Servers](https://tryhackme.com/room/protocolsandservers) | Easy | HTTP/FTP/SMTP insecurities | 📝 |
| [Protocols and Servers 2](https://tryhackme.com/room/protocolsandservers2) | Medium | Password attacks · SSH/TLS | 📝 |

</details>

<details>
<summary><h3>🛠️ Tooling (Burp · Metasploit · Hydra · Python · Wireshark)</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Burp Suite: The Basics](https://tryhackme.com/room/burpsuitebasics) | Info | Web proxy fundamentals | 📝 |
| [Burp Suite: Repeater](https://tryhackme.com/room/burpsuiterepeater) | Info | Request manipulation | 📝 |
| [Burp Suite: Intruder](https://tryhackme.com/room/burpsuiteintruder) | Medium | Automated attacks | 📝 |
| [Burp Suite: Other Modules](https://tryhackme.com/room/burpsuiteom) | Easy | Decoder/Comparer/etc | 📝 |
| [Burp Suite: Extensions](https://tryhackme.com/room/burpsuiteextensions) | Easy | BApp extensions | 📝 |
| [Metasploit: Introduction](https://tryhackme.com/room/metasploitintro) | Easy | MSF framework | 📝 |
| [Hydra](https://tryhackme.com/room/hydra) | Easy | Network login cracking | 📝 |
| [Python Basics](https://tryhackme.com/room/pythonbasics) | Easy | Scripting | 📝 |
| [Python for Pentesters](https://tryhackme.com/room/pythonforcybersecurity) | Easy | Offensive scripting | 📝 |
| [Wireshark: The Basics](https://tryhackme.com/room/wiresharkthebasics) | Easy | Packet analysis | 📝 |

</details>

<details>
<summary><h3>🎯 Red Team & Social Engineering</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Red Team Engagements](https://tryhackme.com/room/redteamengagements) | Easy | Engagement methodology | 📝 |
| [Phishing](https://tryhackme.com/room/phishingyl) | Medium | SET · phishing infra | 📝 |

</details>

<details>
<summary><h3>💥 CVEs & Exploitation</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [NoScope: Finding RCE](https://tryhackme.com/room/noscoperce) | Medium | Alf.io 0-day (CVE-2026-35482) | 📝 |
| [CVE-2026-46300: Fragnesia](https://tryhackme.com/room/cve202646300) | Easy | Linux LPE | 📝 |
| [n8n: CVE-2025-68613](https://tryhackme.com/room/n8ncve202568613) | Easy | n8n exploitation | 📝 |
| [PHP2Shell](https://tryhackme.com/room/cvephp2shell) | Easy | session.upload_progress RCE | 📝 |
| [Brr](https://tryhackme.com/room/brr) | Easy | ICS/SCADA (OT) | 📝 |

</details>

<details>
<summary><h3>📚 Fundamentals · OWASP · Methodology · Blue/GRC</h3></summary>

| Room | Difficulty | Focus | Writeup |
|---|---|---|---|
| [Pentesting Fundamentals](https://tryhackme.com/room/pentestingfundamentals) | Easy | Ethics & methodology | 📝 |
| [Dive Into Pentesting](https://tryhackme.com/room/diveintopentesting) | Easy | Pentest basics | 📝 |
| [Penetration Testing Frameworks](https://tryhackme.com/room/penetrationtestingframeworks) | Easy | OSSTMM/PTES/etc | 📝 |
| [Offensive Security Intro](https://tryhackme.com/room/offensivesecurityintro) | Easy | First hack | 📝 |
| [OWASP Top 10 2025: IAAA Failures](https://tryhackme.com/room/owasptopten2025one) | Easy | A01/A07/A09 | 📝 |
| [OWASP Top 10 2025: Application Design Flaws](https://tryhackme.com/room/owasptopten2025two) | Easy | A02/A03/A06/A10 | 📝 |
| [OWASP Top 10 2025: Insecure Data Handling](https://tryhackme.com/room/owasptopten2025three) | Easy | A04/A05/A08 | 📝 |
| [Understanding Vulnerability Databases](https://tryhackme.com/room/understandingvulnerabilitydatabases) | Easy | CVE/NVD usage | 📝 |
| [Host-Server Configuration Reviews](https://tryhackme.com/room/hostserverconfigurationreviews) | Easy | Hardening | 📝 |
| [HTTP in Detail](https://tryhackme.com/room/httpindetail) | Easy | Web protocol | 📝 |
| [Search Skills](https://tryhackme.com/room/searchskillscS) | Easy | OSINT / research | 📝 |
| [Cyber Kill Chain](https://tryhackme.com/room/cyberkillchain) | Medium | Lockheed Martin model | 📝 |
| [Governance & Regulation](https://tryhackme.com/room/cybergovernanceregulation) | Easy | Policy/frameworks | 📝 |
| [Defensive Security Intro](https://tryhackme.com/room/defensivesecurityintro) | Easy | SOC/DFIR/SIEM intro | 📝 |
| [Careers in Cyber](https://tryhackme.com/room/careersincyber) | Info | Career paths | 📝 |

</details>

---

## 🗂️ Repository structure

```
ctf-writeups/
├── README.md                 # this index
├── _TEMPLATE_writeup.md      # copy this to start a new writeup
├── Domino/
│   └── Readme.md             # ✅ published
├── Marketplace/
│   └── README.md             # ✅ published
├── Watcher/
│   └── README.md             # ✅ published
├── Empline/
│   └── README.md             # ✅ published
├── PickleRick/
│   └── Readme.md             # (next)
└── ...                       # one folder per room, as I publish
```

**Workflow:** one folder per room → each `Readme.md` follows the template (overview, skills, step-by-step, lessons, redacted flags). I add writeups progressively, prioritising the rooms that best show methodology.

---

## ⚖️ Disclaimer

All work here was performed in **legal, authorised lab environments** on [TryHackMe](https://tryhackme.com). These writeups are for educational purposes. Flags are redacted. Nothing here is to be used against systems without explicit permission.
