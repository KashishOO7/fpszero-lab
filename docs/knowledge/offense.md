# Offense — The Red Team Mindset

!!! warning "Ethics First"
    Understanding offensive techniques makes you a better defender. This knowledge is for authorized security testing, CTF competitions, and research only. Unauthorized access to systems is a criminal offense in virtually every jurisdiction.

---

## The Attacker's Mental Model

Offensive security is not about knowing tools. It is about thinking about systems the way their designers didn't:

> *"Every system was designed assuming certain things about how it will be used. A vulnerability is the gap between that assumption and reality."*

### The Attacker's Questions

When looking at any target, a skilled attacker asks:

1. **What is this system's intended behavior?**
2. **What inputs does it accept?**
3. **How does it handle unexpected input?**
4. **What does it trust that it shouldn't?**
5. **What can I control that affects what it does?**

---

## The Penetration Testing Methodology

### Phases

```
Reconnaissance → Scanning → Enumeration → Exploitation → Post-Exploitation → Reporting
```

**Reconnaissance (Recon):**
Gather intelligence without touching the target. OSINT — the internet remembers everything.

- WHOIS data, DNS records, subdomains
- Google dorking: `site:target.com filetype:pdf`, `inurl:admin site:target.com`
- Job postings (reveal tech stack)
- GitHub — leaked credentials, internal code, API keys
- Shodan — internet-connected devices

**Scanning:**
Active probing — you are now generating traffic the target can detect.

- Port scanning (nmap)
- Service version fingerprinting
- Web application fingerprinting (Wappalyzer, WhatWeb)
- Vulnerability scanning (Nikto for web, OpenVAS for network)

**Enumeration:**
Extract specific information from discovered services.

- Web: directory bruteforcing (gobuster, ffuf), parameter fuzzing
- SMB: users, shares, policies
- SNMP: community strings, MIB walking
- LDAP: AD users, groups, computer objects

**Exploitation:**
Use discovered vulnerabilities to gain access.

**Post-Exploitation:**
- Establish persistence
- Lateral movement (move from one system to others)
- Privilege escalation (gain higher-level access)
- Data exfiltration

---

## Attack Surface

Every interface that accepts input is part of the attack surface:

| Interface | Example Attacks |
| :--- | :--- |
| Web application | SQLi, XSS, SSRF, auth bypass |
| API endpoints | Broken object-level authorization, mass assignment |
| File uploads | Webshell upload, path traversal |
| Authentication | Credential stuffing, brute force, password reset flaws |
| Network services | Unpatched services, default credentials, misconfigs |
| Clients | Phishing, malicious files, browser exploits |
| Supply chain | Compromised dependencies, build systems |

---

## Sub-Domains

- [Web Security →](offense/web_sec.md) — OWASP Top 10, injection, logic flaws
- [Malware Analysis →](offense/malware.md) — Persistence, C2, reverse engineering

---

## Tools Overview

| Tool | Category | Use |
| :--- | :--- | :--- |
| nmap | Scanning | Port/service discovery |
| Burp Suite | Web | Proxy, scanner, intruder |
| Metasploit | Exploitation | Framework for known exploits |
| Gobuster/ffuf | Web | Directory/subdomain bruteforce |
| SQLmap | Web | Automated SQL injection |
| Hydra | Auth | Credential bruteforce |
| Impacket | Windows/AD | SMB, Kerberos, NTLM attacks |
| BloodHound | AD | Active Directory attack path mapping |
| Mimikatz | Windows | Credential dumping |

---

## Learning Path

1. [Complete the Cybersecurity Roadmap →](../roadmaps/cybersecurity.md)
2. [Practice with CTFs →](../roadmaps/ctf.md)
3. Read: *The Hacker Playbook* series (Peter Kim)
4. Read: *Penetration Testing* (Georgia Weidman)
5. Lab: [Hack The Box Pro Labs](https://www.hackthebox.com/hacker/pro-labs) — full AD environments
