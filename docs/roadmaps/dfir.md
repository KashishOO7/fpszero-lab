# Digital Forensics & Incident Response (DFIR)

**Goal:** Investigate compromised systems, reconstruct attacker timelines, recover evidence, and contain incidents using repeatable, legally defensible methods.

!!! warning "Legal Context"
    Digital forensics in professional settings involves chain of custody, evidence integrity, and legal admissibility. Understand your jurisdiction's laws. Always get **written authorization** before touching any system that isn't yours.

---

## What DFIR Actually Is

DFIR splits into two disciplines that often overlap in practice:

- **Digital Forensics (DF):** Recovering and analyzing data from devices — disk images, memory dumps, logs, network captures. The goal is reconstruction of events.
- **Incident Response (IR):** The operational process of detecting, containing, eradicating, and recovering from security incidents.

In most roles, you do both: something bad happens, you respond (IR), then you analyze the evidence (DF) to understand what happened and prevent recurrence.

---

## Phase 0 — Prerequisite Knowledge

| Skill | Why It Matters |
| :--- | :--- |
| Linux CLI | Most forensic tools are command-line driven |
| Windows internals | Most victims run Windows; know the registry, event logs, prefetch |
| Networking (TCP/IP, DNS) | Network forensics is a core sub-discipline |
| File systems | FAT32, NTFS, ext4 — understand how data is stored and deleted |

**Start here if needed:**

- [Windows Forensics — TryHackMe Path](https://tryhackme.com/paths)
- [Linux Fundamentals — Linux Journey](https://labex.io/linuxjourney)

---

## Phase 1 — Core Forensics Concepts

### The Forensic Process

```
Identification → Preservation → Collection → Examination → Analysis → Reporting
```

Every step matters. Skipping preservation (making a forensic image before analysis) can make evidence inadmissible.

### Disk Forensics

What you need to know:

- **Forensic imaging** — bit-for-bit copy, never work on originals
- **File system analysis** — inodes, MFT (NTFS), deleted file recovery
- **File carving** — recovering files from unallocated space
- **Metadata analysis** — timestamps (MACB: Modified, Accessed, Changed, Born)
- **Artifacts:** Prefetch, LNK files, Jump Lists, Shellbags, AmCache, Shimcache

**Tools:**

| Tool | Purpose | Cost |
| :--- | :--- | :--- |
| [Autopsy](https://www.autopsy.com/) | Full disk analysis GUI | Free |
| [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager) | Forensic imaging | Free |
| [Sleuth Kit](https://www.sleuthkit.org/) | CLI disk analysis | Free |
| [Recuva](https://www.ccleaner.com/recuva) | File recovery (Windows) | Free |

```bash
# Create a forensic image with dd
sudo dd if=/dev/sdb of=/mnt/evidence/disk.img bs=4M status=progress

# Verify integrity with sha256
sha256sum /dev/sdb > original.hash
sha256sum /mnt/evidence/disk.img > image.hash
diff original.hash image.hash  # Must match
```

### Memory Forensics

RAM holds what disk doesn't: running processes, network connections, encryption keys, passwords.

**Volatility** is the standard tool:

```bash
# Install Volatility 3
pip3 install volatility3

# List running processes from a memory dump
vol -f memory.dmp windows.pslist

# Find network connections
vol -f memory.dmp windows.netstat

# Dump a specific process
vol -f memory.dmp windows.dumpfiles --pid 1337
```

**Resources:**

| Resource | Type |
| :--- | :--- |
| [Volatility 3 Docs](https://volatility3.readthedocs.io/) | Documentation |
| [MemLabs](https://github.com/stuxnet999/MemLabs) | CTF-style memory challenges |
| [Memory Forensics Cheatsheet](https://downloads.volatilityfoundation.org/releases/2.4/CheatSheet_v2.4.pdf) | Reference |

### Log Analysis

Logs are the timeline of what happened. Know where to find them:

| OS | Key Log Locations |
| :--- | :--- |
| Windows | `%SystemRoot%\System32\winevt\Logs\` — Security.evtx, System.evtx, Application.evtx |
| Linux | `/var/log/auth.log`, `/var/log/syslog`, `/var/log/apache2/`, `/var/log/audit/` |
| macOS | `unified log` via `log show` command; `/var/log/` |

**Critical Windows Event IDs:**

| Event ID | Meaning |
| :--- | :--- |
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4648 | Logon with explicit credentials |
| 4663 | Object access (file/folder) |
| 4688 | Process creation |
| 7045 | New service installed |
| 4698 | Scheduled task created |

---

## Phase 2 — Incident Response

### The IR Lifecycle (NIST SP 800-61)

```
Preparation → Detection & Analysis → Containment → Eradication → Recovery → Post-Incident Activity
```

### SIEM & Log Management

You cannot do IR at scale without a SIEM. The free stack:

- **Elastic Stack (ELK):** Elasticsearch + Logstash + Kibana — most widely deployed
- **Wazuh:** Open source SIEM + XDR built on top of ELK
- **Splunk Free:** 500MB/day — enough for a home lab

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Wazuh Documentation](https://documentation.wazuh.com/) | Docs | Free |
| [TryHackMe — SOC Level 1 Path](https://tryhackme.com/paths) | Labs | Free tier |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Labs | Free tier |

### Network Forensics

- Capture and analyze traffic with **Wireshark** and **tcpdump**
- Identify C2 traffic patterns (beaconing, unusual ports, encoded data)
- Analyze DNS for exfiltration (unusually long subdomains, high query volume)
- Reconstruct sessions, extract transferred files

```bash
# Capture traffic on interface eth0
sudo tcpdump -i eth0 -w capture.pcap

# Extract HTTP objects from a PCAP in Wireshark:
# File → Export Objects → HTTP
```

---

## Phase 3 — Specializations

### Malware Analysis

- **Static analysis:** strings, PE headers, imports, hashes
- **Dynamic analysis:** sandbox execution, behavioral observation (Cuckoo, Any.run)
- **Reverse engineering:** IDA Free, Ghidra (NSA-developed, free), x64dbg

See [Malware Knowledge Page →](../knowledge/offense/malware.md)

### Mobile Forensics

See [Mobile Security Roadmap →](mobile.md)

### Cloud Forensics

- AWS: CloudTrail logs, S3 access logs, VPC Flow Logs
- Azure: Activity Log, Azure Monitor, Defender for Cloud
- GCP: Cloud Audit Logs, VPC flow logs

---

## Resources & Platforms

| Resource | Category |
| :--- | :--- |
| [DFIR.training](https://www.dfir.training/) | Curated learning resources |
| [13Cubed YouTube](https://www.youtube.com/c/13cubed) | Video walkthroughs |
| [DFIR Report](https://thedfirreport.com/) | Real incident case studies |
| [CyberDefenders](https://cyberdefenders.org/) | Blue team CTF labs |
| [NIST SP 800-86](https://nvlpubs.nist.gov/nistpubs/Legacy/SP/nistspecialpublication800-86.pdf) | Guide to Integrating Forensics into IR |
| [SANS DFIR Posters](https://www.sans.org/posters/?focus-area=digital-forensics) | Quick reference cheatsheets |
| [awesome-incident-response](https://github.com/meirwah/awesome-incident-response) | Curated IR tools and resources |
| [LetsDefend](https://letsdefend.io/) | SOC analyst training with real alerts |
