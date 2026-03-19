# Networking — From Packets to Protocols

> Every attack travels over a network. Every defense monitors one. Networking is not optional background knowledge — it is the medium.

---

## The OSI Model (Actually Useful Version)

Most explanations of the OSI model are useless because they're just a list of names. Here's what each layer *does* and why it matters for security:

| Layer | Name | What It Does | Security Relevance |
| :--- | :--- | :--- | :--- |
| 7 | Application | HTTP, DNS, SMTP, TLS | Most web vulns live here |
| 6 | Presentation | Encoding, encryption, compression | SSL/TLS stripping, encoding attacks |
| 5 | Session | Manages connections (sessions) | Session hijacking |
| 4 | Transport | TCP/UDP, ports, flow control | Port scanning, firewall rules |
| 3 | Network | IP addressing, routing | IP spoofing, routing attacks |
| 2 | Data Link | MAC addresses, Ethernet frames | ARP spoofing, VLAN hopping |
| 1 | Physical | Bits on wire/radio | Wiretapping, RF attacks |

In practice: **L7 (App) and L4 (Transport) are where most action happens.**

---

## TCP/IP — The Real Story

### IP Addressing

```bash
# An IP address is just a 32-bit number (IPv4)
# 192.168.1.1 → 11000000.10101000.00000001.00000001

# Subnet mask tells you network vs host portion
# 192.168.1.0/24 → first 24 bits = network, last 8 = hosts
# /24 → 256 addresses, 254 usable (0=network, 255=broadcast)

# Private ranges (RFC 1918) — never routed on public internet
# 10.0.0.0/8      → 10.x.x.x
# 172.16.0.0/12   → 172.16.x.x to 172.31.x.x
# 192.168.0.0/16  → 192.168.x.x

# Useful commands
ip addr show
ip route show
cat /etc/resolv.conf   # DNS servers
```

### The TCP Handshake

```
Client                    Server
  |                          |
  |------SYN (seq=100)------>|  "I want to connect, my sequence starts at 100"
  |                          |
  |<--SYN-ACK (seq=200,-----| "OK, my sequence starts at 200, I got your 100"
  |         ack=101)---------|
  |                          |
  |------ACK (ack=201)------>| "Got it, let's go"
  |                          |
  |====== DATA FLOWS ========|
  |                          |
  |------FIN---------------->| "I'm done sending"
  |<---------FIN-ACK---------|
  |------ACK---------------->|
```

**Why this matters for security:**

- **SYN scan (nmap -sS):** Send SYN, never complete handshake → stealthy port scan
- **SYN flood:** Send millions of SYNs, never ACK → exhaust server's connection table (DoS)
- **TCP session hijacking:** Predict sequence numbers to inject into an existing session

### UDP

UDP has no handshake, no reliability, no ordering guarantee. Faster, but no connection state.

Used by: DNS (53), DHCP (67/68), NTP (123), SNMP (161), many VoIP/gaming protocols.

Security relevance: DNS amplification DDoS (small query → large response, spoofed source IP), UDP flooding.

---

## Key Protocols

### DNS — The Internet's Phone Book

```bash
# DNS resolution process for google.com:
# 1. Check local cache → 2. Query /etc/resolv.conf (your DNS server)
# 3. Your DNS asks root nameservers (.)
# 4. Root refers to .com TLD nameservers
# 5. TLD refers to google.com authoritative nameservers
# 6. Authoritative server returns the A record

# Query types
dig google.com A            # IPv4 address
dig google.com MX           # Mail servers
dig google.com NS           # Nameservers
dig google.com TXT          # Text records (SPF, DKIM, verification tokens)
dig -x 8.8.8.8              # Reverse DNS lookup

# Security-relevant queries
dig axfr @ns1.target.com target.com  # Zone transfer (if misconfigured, dumps all DNS records)
```

**DNS Security Issues:**

| Attack | Description |
| :--- | :--- |
| DNS Spoofing / Cache Poisoning | Inject false records into resolver's cache |
| DNS Amplification DDoS | Use open resolvers to amplify traffic at victim |
| DNS Exfiltration | Encode data in DNS queries to bypass firewalls |
| Subdomain Takeover | DNS record points to unclaimed cloud resource |
| Zone Transfer | AXFR dumps entire zone to attacker |

### HTTP/S

```bash
# HTTP request anatomy
GET /login HTTP/1.1
Host: target.com
User-Agent: Mozilla/5.0
Cookie: session=abc123
Content-Type: application/json

# Key headers from a security lens:
# Server: Apache/2.4.50 ← version disclosure
# X-Powered-By: PHP/7.4 ← more fingerprinting
# Set-Cookie: session=x; HttpOnly; Secure; SameSite=Strict  ← security flags
# Content-Security-Policy: default-src 'self'  ← XSS protection
# Strict-Transport-Security: max-age=31536000  ← HSTS

# Status codes that matter
# 200 → OK
# 301/302 → Redirect (check redirect location)
# 401 → Unauthorized (no credentials)
# 403 → Forbidden (have creds but no permission)
# 500 → Server Error (often reveals info in stack traces)
```

### TLS/SSL

TLS encrypts HTTP traffic. Understanding it means knowing when it's broken:

- **SSL/TLS version attacks:** SSLv2, SSLv3, TLS 1.0/1.1 are deprecated; check with `testssl.sh`
- **Weak cipher suites:** RC4, DES, EXPORT ciphers → man-in-the-middle
- **Certificate issues:** Self-signed, expired, wrong hostname, wrong CA
- **Certificate pinning bypass:** Frida/Objection for mobile apps

```bash
# Check TLS configuration
testssl.sh target.com
openssl s_client -connect target.com:443
nmap --script ssl-enum-ciphers -p 443 target.com
```

---

## Network Analysis Tools

### Wireshark

The fundamental network analysis tool. GUI-based.

```
# Useful Wireshark display filters:
http                           # All HTTP traffic
http.request.method == "POST" # POST requests only
ip.addr == 192.168.1.1        # Traffic to/from specific IP
tcp.port == 8080              # Specific port
dns                           # All DNS traffic
tcp.flags.syn == 1 && tcp.flags.ack == 0  # SYN packets (port scan detection)
```

### tcpdump

CLI packet capture — essential for remote/headless environments:

```bash
# Basic capture
sudo tcpdump -i eth0 -w capture.pcap

# Capture with filter
sudo tcpdump -i eth0 host 192.168.1.1 -w capture.pcap
sudo tcpdump -i eth0 port 80 or port 443
sudo tcpdump -i eth0 'tcp[tcpflags] & tcp-syn != 0'  # SYN packets

# Read a PCAP
tcpdump -r capture.pcap -n
tcpdump -r capture.pcap -A   # Print payload in ASCII
```

### nmap

The standard port scanner. Know it deeply.

```bash
# Common scan types
nmap -sS target          # SYN scan (stealthy, requires root)
nmap -sV target          # Version detection
nmap -sC target          # Default scripts
nmap -O target           # OS detection
nmap -A target           # Aggressive: -sV -sC -O --traceroute
nmap -p- target          # All 65535 ports
nmap -p 22,80,443 target # Specific ports

# Network discovery
nmap -sn 192.168.1.0/24  # Ping sweep (no port scan)
nmap -PR 192.168.1.0/24  # ARP ping (LAN only)

# Output formats
nmap -oN out.txt target  # Normal
nmap -oX out.xml target  # XML (for parsing)
nmap -oG out.gnmap target # Grepable

# NSE (Nmap Scripting Engine)
nmap --script vuln target                  # Run all vuln scripts
nmap --script http-enum target -p 80       # Enumerate web paths
nmap --script smb-vuln-ms17-010 target    # Check for EternalBlue
```

---

## Firewalls & Filtering

```bash
# iptables — Linux firewall
iptables -L -v -n                    # List all rules
iptables -A INPUT -p tcp --dport 22 -j ACCEPT
iptables -A INPUT -j DROP            # Default deny

# nftables (modern replacement for iptables)
nft list ruleset

# Check if you're behind NAT or can route
traceroute google.com
ip route get 8.8.8.8
```

---

## References

| Resource | Type |
| :--- | :--- |
| [Wireshark User Guide](https://www.wireshark.org/docs/wsug_html/) | Documentation |
| [nmap.org Reference Guide](https://nmap.org/book/man.html) | Reference |
| [RFC 793 — TCP](https://www.rfc-editor.org/rfc/rfc793) | Original TCP spec |
| [Beej's Guide to Network Programming](https://beej.us/guide/bgnet/) | Free book |
| [Networking Zine — Julia Evans](https://jvns.ca/networking-zine.pdf) | Visual intro |
