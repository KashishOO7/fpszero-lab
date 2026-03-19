# Linux — Internals & Security

> Linux is not just a command line. It is a philosophy of transparency: everything is a file, everything is composable, everything leaves a trace.

---

## The Filesystem Hierarchy

| Path | Purpose |
| :--- | :--- |
| `/` | Root of the entire filesystem |
| `/bin`, `/usr/bin` | Essential user binaries |
| `/sbin`, `/usr/sbin` | System binaries (root-level) |
| `/etc` | Configuration files |
| `/home` | User home directories |
| `/root` | Root user's home |
| `/tmp` | Temporary files (writable by all, cleared on reboot) |
| `/var` | Variable data — logs (`/var/log`), spools, runtime data |
| `/proc` | Virtual filesystem — kernel/process state at runtime |
| `/dev` | Device files (`/dev/sda`, `/dev/null`, `/dev/urandom`) |
| `/sys` | Hardware/driver info from kernel |
| `/lib`, `/usr/lib` | Shared libraries |

### /proc — The Security Goldmine

`/proc` exposes the live state of the kernel and all running processes:

```bash
cat /proc/version          # Kernel version
cat /proc/cpuinfo          # CPU info
cat /proc/meminfo          # Memory usage
cat /proc/net/tcp          # Active TCP connections (hex format)
ls /proc/1234/             # Everything about PID 1234
cat /proc/1234/cmdline     # Command that started process
cat /proc/1234/environ     # Environment variables (may have secrets!)
cat /proc/1234/maps        # Memory map of process
cat /proc/1234/fd/         # Open file descriptors
```

---

## File Permissions Deep Dive

```bash
# Permission bits: rwxrwxrwx (owner | group | others)
# r=4, w=2, x=1

chmod 755 file   # rwxr-xr-x → owner: rwx, group: r-x, other: r-x
chmod 640 file   # rw-r----- → owner: rw, group: r, other: none
chmod +x file    # Add execute for all
chmod u+s file   # Set SUID bit

# Find interesting permissions
find / -perm -4000 2>/dev/null    # SUID files (run as owner)
find / -perm -2000 2>/dev/null    # SGID files (run as group)
find /tmp -type f -perm -o+w 2>/dev/null  # World-writable in /tmp

# Access Control Lists (ACLs) — more granular than chmod
getfacl file
setfacl -m u:username:rwx file
```

### Special Permission Bits

| Bit | Name | Effect |
| :--- | :--- | :--- |
| SUID (4xxx) | Set User ID | File runs with *owner's* permissions |
| SGID (2xxx) | Set Group ID | File runs with *group's* permissions |
| Sticky (1xxx) | Sticky bit | On directories: only owner can delete their own files |

SUID is the most security-relevant. Misconfigured SUID binaries are a common privesc vector:

```bash
# Classic SUID privesc: if /usr/bin/python3 is SUID root:
/usr/bin/python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

---

## Process Management

```bash
# View processes
ps aux                     # All processes, detailed
ps -ef                     # Alternative format
top / htop                 # Real-time process viewer
pstree                     # Process hierarchy

# Process signals
kill -9 PID                # SIGKILL — force terminate
kill -15 PID               # SIGTERM — graceful terminate
kill -1 PID                # SIGHUP — reload config (for daemons)

# Background processes
command &                  # Run in background
jobs                       # List background jobs
fg %1                      # Bring job 1 to foreground
nohup command &            # Run and ignore hangup signal

# Scheduling and priorities
nice -n 10 command         # Lower priority
renice -n 5 -p PID         # Change running process priority
```

### /proc and Process Analysis

```bash
# Investigate a running process
cat /proc/$PID/cmdline | tr '\0' ' '  # Full command
cat /proc/$PID/status                  # State, UIDs, memory
ls -la /proc/$PID/fd/                  # Open files
cat /proc/$PID/net/tcp                 # Network connections

# Find what's listening on a port
ss -tlnp                   # TCP listening sockets with PIDs
ss -ulnp                   # UDP listening
lsof -i :80                # What's using port 80
```

---

## Shell & Text Processing

### Essential One-Liners

```bash
# Find files
find / -name "*.conf" 2>/dev/null
find / -user root -writable 2>/dev/null
find /home -name "*.txt" -mtime -7  # Modified in last 7 days

# Grep patterns
grep -r "password" /etc/ 2>/dev/null
grep -i "error\|fail\|denied" /var/log/auth.log
grep -o '[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}\.[0-9]\{1,3\}' file  # Extract IPs

# AWK — column processing
awk '{print $1}' file              # Print first column
awk -F: '{print $1,$3}' /etc/passwd  # Parse passwd file
awk '$3 >= 1000' /etc/passwd       # UID >= 1000 (normal users)

# SED — stream editor
sed 's/old/new/g' file             # Replace all occurrences
sed -n '10,20p' file               # Print lines 10-20
sed '/^#/d' config.conf            # Remove comment lines

# Sort & count
sort file | uniq -c | sort -rn    # Count occurrences, sort by freq
cat access.log | awk '{print $1}' | sort | uniq -c | sort -rn | head  # Top IPs
```

### Bash Scripting Basics

```bash
#!/bin/bash
# Recon script skeleton

TARGET=$1

echo "[*] Starting recon on $TARGET"

# Check if target is alive
if ping -c 1 "$TARGET" &>/dev/null; then
    echo "[+] Host is alive"
else
    echo "[-] Host appears down"
    exit 1
fi

# Loop example
for port in 22 80 443 8080; do
    if timeout 1 bash -c "echo >/dev/tcp/$TARGET/$port" 2>/dev/null; then
        echo "[+] Port $port is open"
    fi
done
```

---

## Log Analysis

```bash
# Auth logs — failed logins, sudo usage, SSH
tail -f /var/log/auth.log
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
grep "sudo" /var/log/auth.log | grep -v "session"

# Syslog
tail -f /var/log/syslog
journalctl -u ssh --since "1 hour ago"  # Systemd logs

# Web server
tail -f /var/log/apache2/access.log
grep " 500 " /var/log/nginx/error.log   # Server errors
```

---

## Linux Privilege Escalation — Checklist

This is for CTF/lab environments and authorized penetration testing.

```bash
# System info
uname -a; cat /etc/os-release

# Current user context
id; whoami; groups; sudo -l

# Interesting files
cat /etc/passwd | grep -v nologin
cat /etc/cron* 2>/dev/null
ls -la /etc/cron.d/

# SUID/SGID
find / -perm -u=s -o -perm -g=s 2>/dev/null | grep -v proc

# Writable files by others
find / -writable -not -path "/proc/*" -not -path "/sys/*" 2>/dev/null

# Capabilities (alternative to SUID)
getcap -r / 2>/dev/null

# Environment and PATH
env; echo $PATH

# Network
ss -tlnp; ip addr; ip route

# Automated: LinPEAS
# curl -L https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh | bash
```

---

## Key References

| Resource | Type |
| :--- | :--- |
| [Linux Journey](https://labex.io/linuxjourney) | Free book |
| [Linux Privilege Escalation — GTFOBins](https://gtfobins.github.io/) | Privesc techniques |
| [Explainshell](https://explainshell.com/) | Break down any shell command |
| [Over the Wire: Bandit](https://overthewire.org/wargames/bandit/) | Practice |
| [linPEAS](https://github.com/peass-ng/PEASS-ng) | Automated privesc enumeration |
