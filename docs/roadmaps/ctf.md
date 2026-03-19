# CTF & Wargames Roadmap

**Goal:** Build practical security skills through structured, gamified challenges — from beginner wargames to advanced competition CTFs.

!!! info "What is a CTF?"
    A **Capture The Flag** competition presents security challenges where you find a hidden string (a "flag") by exploiting a vulnerability, solving a puzzle, or reversing a binary. Flags prove you solved the challenge.
    Format: `FLAG{something_here}` or `picoCTF{...}` or similar.

---

## Why CTFs?

CTFs teach you to *think like an attacker* in a structured, legal environment. They are the single best way to build hands-on skills quickly. Even professional security researchers play CTFs to sharpen skills they don't use daily.

---

## CTF Categories

| Category | What It Tests |
| :--- | :--- |
| **Web** | XSS, SQLi, SSRF, auth bypass, logic flaws |
| **Pwn / Binary Exploitation** | Buffer overflows, format strings, ROP chains, heap exploitation |
| **Reverse Engineering** | Decompiling binaries, cracking obfuscated code |
| **Cryptography** | Breaking weak ciphers, RSA flaws, hash collisions |
| **Forensics** | Disk/memory/network analysis, steganography |
| **OSINT** | Finding information from public sources |
| **Misc** | Programming challenges, trivia, creative puzzles |

---

## Phase 1 — Start Here (Absolute Beginners)

### Wargames (Level-Based, No Time Pressure)

These are better than competitions for beginners — you go at your own pace.

**OverTheWire: Bandit** — Do this first. Always.

```bash
# Connect to Bandit Level 0
ssh bandit0@bandit.labs.overthewire.org -p 2220
# Password: bandit0
# Goal: find the password for Level 1
# Read: https://overthewire.org/wargames/bandit/bandit1.html
```

Each level teaches a real Linux/security concept. Bandit has 34 levels. Complete all of them before moving on.

| Wargame | Platform | Teaches | Level |
| :--- | :--- | :--- | :--- |
| [Bandit](https://overthewire.org/wargames/bandit/) | OverTheWire | Linux basics | Beginner |
| [Natas](https://overthewire.org/wargames/natas/) | OverTheWire | Web security | Beginner |
| [Leviathan](https://overthewire.org/wargames/leviathan/) | OverTheWire | Linux privesc | Beginner |
| [Krypton](https://overthewire.org/wargames/krypton/) | OverTheWire | Cryptography | Beginner |
| [PicoCTF](https://picoctf.org/) | CMU | All categories | Beginner |
| [UnderTheWire](https://underthewire.tech/) | UTW | PowerShell | Beginner |

### Guided Lab Platforms

| Platform | Why Use It | Cost |
| :--- | :--- | :--- |
| [TryHackMe](https://tryhackme.com) | Guided rooms, walkthroughs, learning paths | Free tier |
| [Hack The Box Academy](https://academy.hackthebox.com) | Structured modules | Free tier |

---

## Phase 2 — Intermediate (Category Specialization)

### Web Security CTF Skills

Master these in PortSwigger Academy (free, lab-based):

- SQL Injection (including blind, time-based, out-of-band)
- Cross-Site Scripting (reflected, stored, DOM)
- CSRF
- XXE (XML External Entity)
- SSRF (Server-Side Request Forgery)
- Insecure Deserialization
- JWT attacks
- OAuth 2.0 flaws

```python
# Simple SQLi test payload
' OR '1'='1
' OR 1=1--
' UNION SELECT NULL,NULL--
' UNION SELECT table_name,NULL FROM information_schema.tables--

# Time-based blind SQLi (MySQL)
' AND SLEEP(5)--
```

### Cryptography CTF Skills

Core math you need:

- **Modular arithmetic** — `a mod n`, `pow(a, b, n)`
- **RSA basics** — `n = p*q`, `e*d ≡ 1 (mod φ(n))`, `m^e mod n`
- **Common RSA attacks:** small e (e=3), common factor, Wiener's attack, LSB oracle
- **Classic ciphers:** Caesar, Vigenere, XOR, substitution
- **Hashing:** MD5/SHA collision properties, hash length extension

```python
# Python crypto toolkit
from Crypto.Util.number import *
from sympy import factorint
import gmpy2

# RSA decryption given p, q, e, c
p, q, e, c = ...
n = p * q
phi = (p-1) * (q-1)
d = pow(e, -1, phi)       # Modular inverse
m = pow(c, d, n)
print(long_to_bytes(m))
```

**Resources:**

| Resource | Type |
| :--- | :--- |
| [Cryptopals](https://cryptopals.com/) | Challenges, teaches real crypto attacks |
| [CryptoHack](https://cryptohack.org/) | Interactive crypto CTF platform |
| [RsaCTFTool](https://github.com/RsaCtfTool/RsaCtfTool) | Automated RSA attack tool |

### Binary Exploitation (Pwn)

This is the steepest learning curve. Prerequisites: C programming, x86/x64 assembly.

```bash
# Essential tools
sudo apt install gdb pwndbg patchelf
pip install pwntools

# Check binary protections
checksec --file=./binary

# Basic pwntools script structure
from pwn import *

p = process('./binary')   # or remote('ip', port)
# p = remote('challs.example.com', 1337)

payload = b'A' * 64      # overflow
payload += p64(0xdeadbeef)  # return address

p.sendline(payload)
p.interactive()
```

| Resource | Type |
| :--- | :--- |
| [pwn.college](https://pwn.college/) | Structured binary exploitation course, free |
| [Exploit Education](https://exploit.education/) | VMs for exploit practice |
| [LiveOverflow YouTube](https://www.youtube.com/c/LiveOverflow) | CTF/pwn video series |

### Forensics CTF Skills

- **Steganography:** Tools — `steghide`, `binwalk`, `zsteg`, `stegsolve`
- **File format analysis:** `file`, `xxd`, `strings`, `exiftool`
- **Network forensics:** Wireshark, `tshark`, `scapy`
- **Memory forensics:** Volatility 3

```bash
# Forensics first-pass checklist for any unknown file
file mystery          # Identify file type
strings mystery       # Extract printable strings
xxd mystery | head -20  # View hex header
binwalk mystery       # Detect embedded files
exiftool mystery      # Extract metadata

# Extract embedded files
binwalk --extract mystery
```

### Reverse Engineering

```bash
# Static analysis tools
strings binary          # Quick string dump
objdump -d binary       # Disassemble
readelf -a binary       # ELF headers

# Dynamic analysis
gdb ./binary
ltrace ./binary         # Library call trace
strace ./binary         # System call trace

# Decompilers (use for complex binaries)
# Ghidra (free, NSA): https://ghidra-sre.org/
# IDA Free: https://hex-rays.com/ida-free/
# Binary Ninja (paid, student discount): https://binary.ninja/
```

---

## Phase 3 — Competition CTFs

Once you can solve beginner/intermediate challenges solo, join competitions:

| Competition | Level | Schedule |
| :--- | :--- | :--- |
| [picoCTF](https://picoctf.org/) | Beginner | Annual (March) |
| [CTFtime.org](https://ctftime.org/) | All levels | Ongoing calendar |
| [HackTheBox CTF](https://ctf.hackthebox.com/) | Intermediate | Recurring |
| [DefCon CTF (Quals)](https://defcon.org/) | Expert | Annual |
| [Google CTF](https://capturetheflag.withgoogle.com/) | Expert | Annual |
| [CSAW CTF](https://www.csaw.io/ctf) | Intermediate | Annual (Sept) |

### CTF Methodology

```
1. Survey all challenges — don't rabbit-hole immediately
2. Sort by solve count — more solves = easier
3. Pick the category you're strongest in first
4. Document your work as you go (you'll need it for the writeup)
5. After 45 mins stuck: check if a hint exists, look for related writeups from older CTFs
6. After the CTF: write up every challenge you solved (even easy ones)
```

---

## CTF Tools Arsenal

```bash
# Install a baseline CTF toolkit on Kali/Ubuntu
sudo apt install -y binwalk steghide foremost exiftool pwndbg \
  python3-pip nmap gobuster ffuf john hashcat

pip3 install pwntools requests beautifulsoup4 pycryptodome gmpy2

# Useful online tools
# CyberChef — https://gchq.github.io/CyberChef/ (Swiss army knife)
# dCode — https://www.dcode.fr/ (cipher identification + solvers)
# RevShells — https://www.revshells.com/ (reverse shell generator)
# GTFOBins — https://gtfobins.github.io/ (Linux privesc)
# LOLBAS — https://lolbas-project.github.io/ (Windows privesc)
```

---

## CTF Writeup Resources

Reading others' writeups is one of the best learning methods:

- [CTFtime writeups](https://ctftime.org/writeups)
- [0x00sec](https://0x00sec.org/)
- [HackTricks](https://book.hacktricks.wiki/en/) — massive collection of techniques
- [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings)

---

!!! tip "The Meta-Skill"
    CTF success is 20% knowing the tools and 80% **being systematic**. Keep notes. Build personal checklists for each category. After every CTF, add what you learned to your notes. After 6 months, your personal reference will be worth more than any cheatsheet.
