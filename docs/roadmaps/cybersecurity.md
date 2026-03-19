# Cybersecurity — Zero to Analyst

**Goal:** Build the foundational knowledge and practical skills to work in cybersecurity — as an analyst, researcher, or specialist in any sub-domain.

!!! warning "Ethical Use"
    This roadmap teaches you how attackers think so defenders can be better. All practice must happen in **authorized environments** — your own machines, lab VMs, or legal platforms like TryHackMe and HackTheBox. Unauthorized access is a crime.

---

## Phase 0 — Prerequisites

Before touching any security tool, you need these foundations. If you already have them, skip ahead.

### Computer Networking

Security is applied networking. Start here.

- **OSI Model** — understand each layer's function, not just the names
- **TCP/IP** — how packets travel, what a handshake actually does at the bit level
- **DNS, HTTP/S, SMTP, FTP, SSH** — know these protocols cold
- **Subnetting & CIDR** — practice until /24, /16, /8 are instant in your head
- **Wireshark basics** — read a PCAP and tell a story about what happened

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Professor Messer CompTIA Network+](https://www.professormesser.com/network-plus/n10-008/n10-008-video/n10-008-training-course/) | Video | Free |
| [Cisco Networking Basics](https://skillsforall.com/course/networking-basics) | Course | Free |
| [Wireshark for Beginners — Udemy](https://www.udemy.com/course/wireshark-tutorial/) | Video | Paid (watch for sales) |
| [Subnetting Practice](https://subnettingpractice.com/) | Interactive | Free |

### Linux Fundamentals

The command line is your primary interface in security work.

- Navigate the filesystem (`cd`, `ls`, `find`, `locate`)
- File permissions (`chmod`, `chown`, `umask`)
- Process management (`ps`, `top`, `kill`, `systemctl`)
- Text processing (`grep`, `awk`, `sed`, `cut`)
- Scripting basics — write a simple bash script

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [OverTheWire: Bandit](https://overthewire.org/wargames/bandit/) | Wargame | Free |
| [Linux Journey](https://labex.io/linuxjourney) | Book | Free |
| [Linux Journey](https://labex.io/linuxjourney) | Interactive | Free |

### Python Basics

You need to be able to read, modify, and write simple scripts.

- Variables, loops, conditionals
- File I/O
- Requests library — making HTTP calls in code
- Parsing JSON

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Python for Everybody](https://www.py4e.com/) | Course | Free |
| [Automate the Boring Stuff](https://automatetheboringstuff.com/) | Book | Free |

---

## Phase 1 — Security Foundations

### Core Security Concepts

- CIA Triad (Confidentiality, Integrity, Availability)
- Authentication vs Authorization vs Accounting (AAA)
- Cryptography basics: symmetric vs asymmetric, hashing, PKI
- Vulnerability vs Exploit vs Payload — know the difference
- Attack surface and threat modeling

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Security+ Study Guide — Professor Messer](https://www.professormesser.com/security-plus/sy0-701/sy0-701-video/sy0-701-comptia-security-plus-course/) | Video | Free |
| [Cybrary Intro to IT & Cybersecurity](https://www.cybrary.it/course/intro-to-it-cybersecurity/) | Course | Free |
| [TryHackMe — Pre-Security Path](https://tryhackme.com/paths) | Labs | Free tier |

### Hands-On Lab Setup

Set up your own lab — this is non-negotiable.

```bash
# Recommended base setup
# 1. Install VirtualBox or VMware Workstation Player (free)
# 2. Download Kali Linux ISO → https://www.kali.org/get-kali/
# 3. Download a vulnerable target VM:
#    - Metasploitable 2: https://sourceforge.net/projects/metasploitable/
#    - DVWA: https://github.com/digininja/DVWA
# 4. Create a host-only network in VirtualBox
#    so your VMs can talk but can't reach the internet
```

---

## Phase 2 — Choose a Lane

Security is broad. After foundations, **specialize**. You can always branch out later.

### Lane A: Web Security (Bug Bounty / Pentesting)

Focus on OWASP Top 10, then go deeper:

- **Injection attacks** — SQLi, command injection, LDAP injection
- **Broken authentication** — session hijacking, credential stuffing
- **XSS** — reflected, stored, DOM-based
- **IDOR & Access Control flaws**
- **SSRF & XXE**
- **Business logic bugs** — the ones scanners miss

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security) | Labs | Free |
| [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) | Guide | Free |
| [Hacker101](https://www.hacker101.com/) | Course + CTF | Free |
| [Bug Bounty Bootcamp (book)](https://nostarch.com/bug-bounty-bootcamp) | Book | Paid |

**Practice Platforms:**

- [HackTheBox](https://hackthebox.com) — retired machines
- [TryHackMe](https://tryhackme.com) — guided rooms
- [PentesterLab](https://pentesterlab.com)

### Lane B: Digital Forensics & Incident Response (DFIR)

See the [full DFIR roadmap →](dfir.md)

### Lane C: CTF & Competitive Hacking

See the [full CTF roadmap →](ctf.md)

---

## Phase 3 — Certifications (Entirely Optional)

Certs are not a replacement for skill. You can always skip these and demonstrate your abilities through real work, writeups, and contributions instead. That said, some recruiters and HR filters still look for them.

| Cert | Vendor | Level | Latest Pricing |
| :--- | :--- | :--- | :--- |
| CompTIA Security+ | CompTIA | Beginner | [Search current price](https://duckduckgo.com/?q=CompTIA+Security%2B+exam+price+2025) |
| eJPT | INE Security | Beginner | [Search current price](https://duckduckgo.com/?q=eJPT+certification+price+2025) |
| CompTIA PenTest+ | CompTIA | Intermediate | [Search current price](https://duckduckgo.com/?q=CompTIA+PenTest%2B+exam+price+2025) |
| OSCP | OffSec | Advanced | [Search current price](https://duckduckgo.com/?q=OSCP+certification+price+2025) |
| CySA+ | CompTIA | Intermediate | [Search current price](https://duckduckgo.com/?q=CompTIA+CySA%2B+exam+price+2025) |

!!! tip "If you do pursue certs"
    **eJPT → OSCP** is the practical path for offensive security.
    **Security+ → CySA+** is the path for defensive/analyst roles.
    But a strong portfolio of real work speaks louder than any acronym.

---

## Phase 4 — Build a Portfolio

Employers want to see you *do* things, not just know things.

- Document your CTF writeups on a blog or GitHub
- Write a tool (even a small one) — a port scanner, a log parser, a hash cracker
- Contribute to an open-source security project
- Submit a bug bounty report (even a low-severity one counts)
- Do a mock penetration test report on a lab environment

---

## Continuous Learning

Security is not a destination. Stay current:

- [Krebs on Security](https://krebsonsecurity.com/) — threat intelligence
- [The Hacker News](https://thehackernews.com/) — news
- [Exploit-DB](https://www.exploit-db.com/) — live exploits
- [NVD / CVE Database](https://nvd.nist.gov/) — vulnerability tracking
- [SANS Internet Storm Center](https://isc.sans.edu/) — daily threat briefings
