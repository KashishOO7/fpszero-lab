# Digital Forensics & Incident Response

> Forensics is archaeology with a ticking clock. You are reconstructing what happened from the traces left behind — and those traces degrade or get overwritten every minute the system stays running.

See the [full DFIR Roadmap →](../../roadmaps/dfir.md) for the learning path.
This page is the reference synthesis — the condensed operational knowledge.

---

## The Forensic Mindset

Two principles govern everything:

**1. Preserve before you analyze.**
Never work on originals. Create a forensic image, verify its hash, work on the copy. This protects evidence integrity and ensures reproducibility.

**2. Collect most volatile first.**
Data in RAM disappears when power is cut. Network state disappears when connections drop. Disk artifacts persist longer. Always collect in this order:

```
RAM → Network state → Running processes → Open files → Disk image → Logs
```

---

## Acquisition

### Memory Acquisition

**Linux:** LiME (Loadable Kernel Module) — [GitHub](https://github.com/jtsylve/LiME)
**Windows:** WinPMem ([GitHub](https://github.com/Velocidex/WinPmem)), DumpIt, or Magnet RAM Capture (free GUI)
**macOS:** osxpmem (older systems); note that macOS SIP may prevent acquisition on modern hardware

### Disk Imaging

**Tools:** `dd` (standard, works everywhere), `dcfldd` (dd with built-in hashing), `ewfacquire` (Expert Witness Format), FTK Imager (Windows GUI, free). Always verify integrity by comparing source and image hashes before analysis.

---

## Windows Artifacts Reference

### Registry Forensics

The registry is a goldmine for understanding what happened.

```
Key artifact locations (in registry hives):

PERSISTENCE:
HKCU\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\Software\Microsoft\Windows\CurrentVersion\Run
HKLM\System\CurrentControlSet\Services  (services)
HKCU\Software\Microsoft\Windows NT\CurrentVersion\Winlogon

USER ACTIVITY:
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RunMRU  (run dialog history)
NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\TypedPaths
NTUSER.DAT\Software\Microsoft\Internet Explorer\TypedURLs

SYSTEM INFO:
HKLM\System\CurrentControlSet\Control\ComputerName
HKLM\System\CurrentControlSet\Control\TimeZoneInformation
HKLM\System\CurrentControlSet\Services\Tcpip\Parameters\Interfaces  (network config)

EVIDENCE OF EXECUTION:
HKLM\System\CurrentControlSet\Control\Session Manager\AppCompatCache  (Shimcache)
HKCU\Software\Microsoft\Windows\CurrentVersion\Explorer\UserAssist
NTUSER.DAT\Software\Classes\Local Settings\Software\Microsoft\Windows\Shell\MuiCache
```

```bash
# Parse registry hives with regipy (Python)
pip install regipy

# Or use RegRipper (Perl, many plugins)
# https://github.com/keydet89/RegRipper3.0
rip.pl -r NTUSER.DAT -p ntuser
rip.pl -r SYSTEM -p shimcache
```

### Windows Prefetch

Prefetch files record program execution. Location: `C:\Windows\Prefetch\*.pf`

```bash
# Parse with PECmd (Eric Zimmermann's tool — free)
PECmd.exe -d "C:\Windows\Prefetch" --csv output\

# Each .pf file records:
# - Program path and name
# - Last run time (up to 8 timestamps in Win10)
# - Run count
# - Files loaded during execution (libraries, config files)
```

### Event Log Analysis

```bash
# Export and parse with EvtxECmd (Eric Zimmermann's tool)
EvtxECmd.exe -d "C:\Windows\System32\winevt\Logs" --csv output\

# Or with Python
pip install python-evtx
python3 -c "
import Evtx.Evtx as evtx
import Evtx.Views as e_views
with evtx.Evtx('Security.evtx') as log:
    for record in log.records():
        print(record.xml())
" | grep -A5 'EventID>4624'

# Quick hunting with chainsaw (fast Sigma-based event log scanner)
# https://github.com/WithSecureLabs/chainsaw
chainsaw hunt C:\Windows\System32\winevt\Logs --sigma sigma-rules/ --mapping mappings/sigma-event-logs-all.yml
```

### Browser Artifacts

| Browser | Location | Artifacts |
| :--- | :--- | :--- |
| Chrome | `%LOCALAPPDATA%\Google\Chrome\User Data\Default\` | History, Cookies, Login Data, Cache |
| Firefox | `%APPDATA%\Mozilla\Firefox\Profiles\*.default\` | places.sqlite, cookies.sqlite, logins.json |
| Edge | `%LOCALAPPDATA%\Microsoft\Edge\User Data\Default\` | Same structure as Chrome |

```bash
# Chrome History is a SQLite database
sqlite3 "History" "SELECT url, title, datetime(last_visit_time/1000000-11644473600,'unixepoch','localtime') FROM urls ORDER BY last_visit_time DESC LIMIT 50;"

# Hindsight — cross-browser history parser
pip install hindsight
hindsight.py -i "C:\Users\User\AppData\Local\Google\Chrome\User Data\Default" -o report
```

---

## Memory Analysis with Volatility 3

```bash
# Install
pip3 install volatility3

# Profile detection (usually automatic)
vol -f memory.raw windows.info

# Core plugins
vol -f memory.raw windows.pslist          # Running processes
vol -f memory.raw windows.pstree         # Process hierarchy (spot injections)
vol -f memory.raw windows.cmdline        # Command lines of processes
vol -f memory.raw windows.netstat        # Network connections
vol -f memory.raw windows.netscan        # More thorough network scan
vol -f memory.raw windows.filescan       # Files open in memory
vol -f memory.raw windows.dlllist        # DLLs loaded by processes
vol -f memory.raw windows.malfind       # Find injected code (anomalous VAD entries)
vol -f memory.raw windows.hashdump      # Extract NTLM hashes (if SYSTEM level)

# Malware-hunting plugins
vol -f memory.raw windows.malfind        # Injected memory regions
vol -f memory.raw windows.hollowfind    # Process hollowing detection
vol -f memory.raw windows.svcscan       # Services (including hidden)
vol -f memory.raw windows.handles       # Object handles per process

# Dump specific process memory
vol -f memory.raw windows.dumpfiles --pid 1234

# Extract a specific DLL
vol -f memory.raw windows.dlllist --pid 1234
vol -f memory.raw windows.dumpfiles --pid 1234 --virtaddr 0x7ff...
```

### Indicators of Compromise in Memory

```bash
# Processes to investigate:
# - Parent/child relationships that don't make sense
#   (e.g., Word.exe spawning cmd.exe or powershell.exe)
# - svchost.exe NOT parented by services.exe
# - Multiple instances of processes that should be unique
# - Processes with names similar to legit ones (svchost vs svch0st)
# - Processes running from temp directories or user profiles

# Check process parent relationships
vol -f memory.raw windows.pstree

# Look for unusual loaded DLLs
vol -f memory.raw windows.dlllist --pid [suspicious_pid]

# malfind finds injected shellcode
vol -f memory.raw windows.malfind --dump --output-dir ./dumped/
# → Then scan dumped regions with AV or YARA
```

---

## Network Forensics

```bash
# Capture traffic
sudo tcpdump -i eth0 -w capture.pcap
# Capture on a mirror port if investigating a remote host

# Parse PCAP statistics
capinfos capture.pcap
tshark -r capture.pcap -q -z io,phs   # Protocol hierarchy

# Extract HTTP objects (files transferred over HTTP)
# Wireshark: File → Export Objects → HTTP

# tshark one-liners
tshark -r capture.pcap -T fields -e ip.src -e ip.dst -e tcp.dstport | sort | uniq -c | sort -rn
tshark -r capture.pcap -Y "http.request" -T fields -e http.host -e http.request.uri
tshark -r capture.pcap -Y "dns" -T fields -e dns.qry.name | sort | uniq -c | sort -rn

# NetworkMiner — passive network forensics, extracts files, credentials, sessions
# https://www.netresec.com/?page=NetworkMiner (free version available)

# zeek (formerly Bro) — turns PCAP into structured logs
zeek -r capture.pcap
# Produces: conn.log, dns.log, http.log, ssl.log, files.log, etc.
```

---

## Linux Forensics

```bash
# Key locations
/var/log/auth.log          # SSH, sudo, su activity
/var/log/syslog            # General system events
/var/log/apache2/          # Web server logs
/etc/passwd                # User accounts
/etc/cron*                 # Scheduled tasks
/tmp/ and /var/tmp/        # Dropper staging areas
~/.bash_history            # User command history
~/.ssh/authorized_keys     # Backdoor SSH keys
/etc/crontab               # System cron jobs

# Find recently modified files (last 7 days)
find / -mtime -7 -type f 2>/dev/null | grep -v /proc | grep -v /sys

# Find SUID files (persistence / privesc)
find / -perm -u=s -type f 2>/dev/null

# Check for unusual cron jobs
crontab -l
ls -la /etc/cron*
cat /etc/crontab

# Forensic timeline with mactime (from Sleuth Kit)
fls -r -m / disk.img > bodyfile.txt
mactime -b bodyfile.txt -d > timeline.csv
```

---

## Tools Summary

| Tool | Platform | Purpose | Cost |
| :--- | :--- | :--- | :--- |
| [Volatility 3](https://github.com/volatilityfoundation/volatility3) | All | Memory analysis | Free |
| [Autopsy](https://www.autopsy.com/) | All | Disk forensics GUI | Free |
| [Sleuth Kit](https://www.sleuthkit.org/) | All | CLI disk forensics | Free |
| [FTK Imager](https://www.exterro.com/digital-forensics-software/ftk-imager) | Windows | Imaging + preview | Free |
| [Eric Zimmermann Tools](https://ericzimmerman.github.io/) | Windows | Registry, event logs, prefetch | Free |
| [KAPE](https://www.kroll.com/kape) | Windows | Artifact collection | Free |
| [Chainsaw](https://github.com/WithSecureLabs/chainsaw) | Windows | Event log triage | Free |
| [Hindsight](https://github.com/obsidianforensics/hindsight) | All | Browser forensics | Free |
| [NetworkMiner](https://www.netresec.com/?page=NetworkMiner) | Windows | Network forensics | Free |
| [Zeek](https://zeek.org/) | Linux | Network traffic analysis | Free |

---

## References

| Resource | Type |
| :--- | :--- |
| [SANS DFIR Cheatsheets](https://www.sans.org/posters/?focus-area=digital-forensics) | Quick reference |
| [13Cubed YouTube](https://www.youtube.com/c/13cubed) | Forensic video walkthroughs |
| [CyberDefenders.org](https://cyberdefenders.org/) | Blue team CTF labs |
| [The Art of Memory Forensics (book)](https://www.amazon.com/Art-Memory-Forensics-Detecting-Malware/dp/1118825098) | Deep memory analysis |
| [Digital Forensics with Open Source Tools (book)](https://www.elsevier.com/books/digital-forensics-with-open-source-tools/altheide/978-1-59749-586-8) | Practical guide |
