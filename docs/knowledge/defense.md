# Defense — Blue Team Operations

> Offense has the initiative. Defense has the visibility. The game is whether you can turn visibility into detection faster than the attacker can achieve their objective.

---

## The Defender's Mental Model

Attackers need to do many things right. Defenders need to catch them doing *one* thing wrong.

This is the **defender's asymmetry** — and it only holds if you have:

1. **Visibility** — logs from the right sources, at the right fidelity
2. **Detection** — rules/anomalies that fire on attacker behavior
3. **Response** — a process to contain and eradicate faster than damage spreads

Most organizations fail at #1. They collect logs. They don't have the *right* logs.

---

## The MITRE ATT&CK Framework

ATT&CK is a matrix of adversary tactics and techniques observed in the wild. It is the shared language of blue team work.

**14 Tactics (the "why"):**

| Tactic | Description |
| :--- | :--- |
| Reconnaissance | Info gathering before attack |
| Resource Development | Building infrastructure |
| Initial Access | Getting into the network |
| Execution | Running code |
| Persistence | Maintaining foothold |
| Privilege Escalation | Gaining higher permissions |
| Defense Evasion | Avoiding detection |
| Credential Access | Stealing credentials |
| Discovery | Learning the environment |
| Lateral Movement | Moving through the network |
| Collection | Gathering target data |
| Command & Control | Communicating with compromised systems |
| Exfiltration | Stealing data out |
| Impact | Destroy, encrypt, disrupt |

Each tactic has dozens of **techniques** with detection notes and mitigations.

**Use it:** [attack.mitre.org](https://attack.mitre.org) — map your detection coverage to this matrix. The gaps are your blind spots.

---

## Logging Strategy

### What to Collect (Priority Order)

**Tier 1 — Critical:**

| Source | Why |
| :--- | :--- |
| Windows Security Event Log | Auth, process creation, object access |
| DNS query logs | C2 detection, data exfiltration |
| Firewall/proxy logs | Network connections, blocked traffic |
| EDR telemetry | Process trees, file activity, memory |
| Authentication logs | AD, VPN, cloud SSO |

**Tier 2 — Important:**

| Source | Why |
| :--- | :--- |
| Web server logs | Attack attempts, enumeration |
| Email gateway logs | Phishing, malicious attachments |
| Cloud API logs | AWS CloudTrail, Azure Activity Log |
| DHCP logs | Correlate IP to host at time of incident |
| Network flow (NetFlow/IPFIX) | Traffic volume anomalies, lateral movement |

### Critical Windows Event IDs

```
Authentication & Account Activity:
4624  → Successful logon (note: Logon Type matters)
4625  → Failed logon
4648  → Logon using explicit credentials (runas, PtH)
4672  → Special privileges assigned (admin logon)
4720  → User account created
4732  → Member added to security-enabled local group
4756  → Member added to security-enabled universal group

Process Activity:
4688  → Process creation (enable command line logging too!)
4689  → Process termination

Object Access:
4663  → Object accessed (file, registry key)
4698  → Scheduled task created ← common persistence
4699  → Scheduled task deleted

Service Activity:
7034  → Service crashed unexpectedly
7036  → Service started/stopped
7045  → New service installed ← watch this

PowerShell:
4103  → Module logging
4104  → Script block logging ← enable this
```

```powershell
# Enable PowerShell Script Block Logging via Group Policy
# Computer Configuration → Administrative Templates → Windows Components
# → Windows PowerShell → Turn on PowerShell Script Block Logging → Enabled

# Or via registry
reg add "HKLM\SOFTWARE\Policies\Microsoft\Windows\PowerShell\ScriptBlockLogging" /v "EnableScriptBlockLogging" /t REG_DWORD /d 1
```

---

## SIEM — Security Information & Event Management

A SIEM collects logs from across your environment, normalizes them, and lets you search and alert.

### The Free Stack

```
Agents/Shippers → Log Aggregation → Search/Analytics → Alerting
(Filebeat/Winlogbeat) → (Elasticsearch) → (Kibana) → (ElastAlert)
```

**Wazuh** is the easiest starting point — it bundles most of this:

```bash
# Deploy Wazuh all-in-one (4GB RAM minimum)
curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh
sudo bash wazuh-install.sh -a
# Access: https://localhost (default: admin/admin — change immediately)
```

**Elastic Stack (ELK) manually:**

```yaml
# docker-compose.yml snippet for development
version: '3'
services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.11.0
    environment:
      - discovery.type=single-node
      - xpack.security.enabled=false
    ports: ["9200:9200"]
  
  kibana:
    image: docker.elastic.co/kibana/kibana:8.11.0
    ports: ["5601:5601"]
```

---

## Detection Engineering

Detection rules are the heart of blue team work. Write them in **Sigma** (vendor-agnostic) and convert.

### Sigma Rules

```yaml
# Example: Detect scheduled task creation via schtasks.exe
title: Suspicious Scheduled Task Creation
id: 92626f28-32aa-41b1-b7d7-a4e97f3b86f9
status: test
description: Detects creation of scheduled tasks via schtasks.exe
author: fpszero
date: 2025/01/01
tags:
    - attack.persistence
    - attack.t1053.005   # MITRE technique
logsource:
    category: process_creation
    product: windows
detection:
    selection:
        Image|endswith: '\schtasks.exe'
        CommandLine|contains:
            - '/create'
            - '/sc'
    condition: selection
falsepositives:
    - Legitimate software installation
    - System administration
level: medium
```

```bash
# Convert Sigma to Splunk/Elastic/QRADAR/etc
pip install sigma-cli
sigma convert -t splunk rule.yml
sigma convert -t es-ql rule.yml   # Elasticsearch
```

### Detection Categories

| Category | Examples |
| :--- | :--- |
| **Signature** | Known malware hashes, IOC matching |
| **Behavioral** | Schtasks created by Office process, LSASS access from unusual process |
| **Statistical** | Beacon detection (periodic C2 intervals), unusually high DNS query volume |
| **Threshold** | >10 failed logins in 5 minutes (brute force) |
| **Correlation** | Successful login after multiple failures from same IP |

---

## Threat Hunting

Proactive search for attacker activity not caught by existing detections.

### Hunting Process

```
1. Hypothesis — "I believe an attacker has established persistence"
2. Data source — Windows Security Event Log (4698: scheduled tasks)
3. Query
4. Triage findings
5. Document (new detection or confirm nothing found)
```

### Sample Hunt Queries (Elastic KQL)

```
// Scheduled task creation outside business hours
event.code: "4698" and NOT hour_of_day:[8 TO 18]

// PowerShell with encoded command (common evasion)
process.name: "powershell.exe" and process.command_line: "*-EncodedCommand*"

// LSASS memory access (credential dumping)
event.code: "10" and target.image: "*lsass.exe"

// Long DNS queries (potential DNS exfiltration)
dns.question.name.length > 50

// Lateral movement: new admin shares
event.code: "5145" and winlog.event_data.ShareName: ("C$" OR "ADMIN$" OR "IPC$")
```

---

## Incident Response Playbook

### Template: Ransomware Response

```
Phase 1: DETECT & TRIAGE (0-30 min)
□ Confirm alert is real (not false positive)
□ Identify affected systems
□ Determine blast radius estimate

Phase 2: CONTAIN (30-120 min)
□ Isolate affected systems from network (DO NOT power off — preserve forensic evidence)
□ Disable compromised accounts
□ Block identified C2 infrastructure at firewall/DNS
□ Preserve logs (collect before they rotate)

Phase 3: INVESTIGATE (parallel with containment)
□ Acquire memory images of affected systems
□ Acquire disk images
□ Collect relevant Windows event logs (Security, System, PowerShell)
□ Identify Patient Zero (first compromised system)
□ Identify attack vector (phishing? RDP? vuln exploitation?)
□ Map lateral movement

Phase 4: ERADICATE
□ Remove malware artifacts (based on investigation findings)
□ Reset all potentially compromised credentials
□ Patch the exploited vulnerability

Phase 5: RECOVER
□ Restore from clean backups (verify backup integrity first)
□ Harden systems based on lessons learned

Phase 6: POST-INCIDENT
□ Write incident report
□ Update detection rules for this attack pattern
□ Conduct lessons learned meeting
□ Update playbooks
```

---

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [MITRE ATT&CK](https://attack.mitre.org) | Framework | Free |
| [Sigma Rules Repository](https://github.com/SigmaHQ/sigma) | Detection rules | Free |
| [TryHackMe SOC Level 1](https://tryhackme.com/paths) | Training | Free tier |
| [Blue Team Labs Online](https://blueteamlabs.online/) | Labs | Free tier |
| [CyberDefenders](https://cyberdefenders.org/) | DFIR challenges | Free |
| [SANS Blue Team Summit Talks](https://www.youtube.com/c/SANSInstitute) | Videos | Free |
| [The Practice of Network Security Monitoring (book)](https://nostarch.com/nsm) | Book | Paid |
