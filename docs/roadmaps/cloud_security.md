# Cloud Security Roadmap

**Goal:** Understand how cloud infrastructure is attacked and defended — covering AWS, GCP, and Azure from an offensive and defensive perspective.

!!! warning "Authorization Required"
    All testing must be in environments you own or have explicit written permission to test.
    Cloud providers have ToS against unauthorized security testing. Use dedicated lab accounts,
    never test production environments without a signed scope agreement.

---

## Why Cloud Security Is Different

The cloud changes the threat model in fundamental ways:

| Traditional | Cloud |
| :--- | :--- |
| Perimeter = firewall | Perimeter = IAM policy |
| Attacker needs network access | Attacker needs one valid credential |
| Misconfiguration is local | Misconfiguration is internet-facing by default |
| Asset inventory is manual | Assets spin up and down in seconds |
| Logging is optional | Logging is built in (if you enable it) |

The cloud is not inherently less secure. It is **transparent** — misconfigurations that would be invisible on-premises are discoverable by anyone with an internet connection and the right query.

---

## Phase 0 — Prerequisites

Before cloud security makes sense, you need:

- Linux fundamentals — you will live in a shell
- Networking basics — VPCs, subnets, security groups are just abstractions over TCP/IP
- Python — for scripting cloud API calls
- Basic security concepts — IAM, least privilege, defense in depth

---

## Phase 1 — Cloud Fundamentals

Pick **AWS first**. It has the largest market share, the most tooling, and the best learning resources. Once you understand the patterns, GCP and Azure follow the same mental model.

### Core AWS Concepts

**Identity & Access Management (IAM) — Learn This First**

IAM is the center of gravity for AWS security. Every attack and every defense in the cloud is ultimately about IAM.

```
IAM Hierarchy:
  Root Account (don't use this)
  └── IAM Users (long-term credentials — minimize)
      └── IAM Roles (temporary credentials — prefer)
          └── IAM Policies (JSON documents defining permissions)
              └── Actions (what you can do: s3:GetObject, ec2:DescribeInstances)
              └── Resources (what you can do it to: arn:aws:s3:::bucket-name/*)
              └── Conditions (when it applies: MFA required, IP range)
```

```json
// Example IAM Policy — Read-only S3 access to one bucket
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": ["s3:GetObject", "s3:ListBucket"],
      "Resource": [
        "arn:aws:s3:::my-bucket",
        "arn:aws:s3:::my-bucket/*"
      ]
    }
  ]
}
```

**Key Services to Understand**

| Service | What It Is | Security Relevance |
| :--- | :--- | :--- |
| **IAM** | Identity & access control | The entire attack surface |
| **S3** | Object storage | Most common misconfiguration (public buckets) |
| **EC2** | Virtual machines | Instance metadata service (SSRF target) |
| **Lambda** | Serverless functions | Environment variable secrets, SSRF |
| **RDS** | Managed databases | Public snapshots, overly permissive SGs |
| **VPC** | Virtual network | Security groups, NACLs, flow logs |
| **CloudTrail** | API call logging | Primary audit log — must be enabled |
| **CloudWatch** | Metrics and alerts | Detection capability |
| **KMS** | Key management | Encryption at rest |
| **Secrets Manager** | Secrets storage | Better than env vars |
| **GuardDuty** | Threat detection | AI-based anomaly detection |

**Setup:**

```bash
# Install AWS CLI
pip install awscli
# Or: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

# Configure with your access keys (for lab account only)
aws configure
# → Access Key ID, Secret Access Key, Region (e.g., us-east-1), Output (json)

# Test
aws sts get-caller-identity
# Returns: Account, UserId, Arn — who you are

# List your S3 buckets
aws s3 ls

# Check your current permissions (what can I do?)
aws iam get-user
aws iam list-attached-user-policies --user-name your-username
```

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [AWS Free Tier](https://aws.amazon.com/free/) | Lab environment | Free (12 months) |
| [AWS Well-Architected Security Pillar](https://aws.amazon.com/architecture/well-architected/?wa-lens-whitepapers.sort-by=item.additionalFields.sortDate&wa-lens-whitepapers.sort-order=desc) | Framework | Free |
| [Cloud Practitioner (CLF-C02)](https://aws.amazon.com/certification/certified-cloud-practitioner/) | Cert | ~$100 |
| [A Cloud Guru / Pluralsight](https://www.pluralsight.com/cloud-guru) | Courses | Paid |

---

## Phase 2 — Cloud Attack Techniques

### The Cloud Attack Lifecycle

```
Recon → Initial Access → Privilege Escalation → Lateral Movement → Exfiltration → Persistence
  ↓            ↓                  ↓                    ↓                ↓              ↓
Shodan,     Phishing,         IAM role          Cross-account       S3, data     Backdoor
bucket      leaked keys,      abuse,            access, assume      APIs         IAM users,
enum,       SSRF →            policy            role, Lambda                     Lambda
GitHub      metadata          misconfig         pivot                            triggers
```

### Common Attack Vectors

**1. Exposed AWS Credentials**

The most common initial access vector. Credentials appear in:

```bash
# GitHub (use GitHub's secret scanning or search manually)
# Search: "AKIA" (AWS access key prefix) in GitHub
# AWS access keys always start with AKIA (user) or ASIA (session)

# Config files accidentally committed
~/.aws/credentials
.env files
docker-compose.yml
CI/CD pipeline configs (GitHub Actions, Jenkins)

# Docker images
docker history image:tag --no-trunc | grep -i aws

# EC2 instance metadata (SSRF → metadata service)
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/
# Returns: temporary credentials for the EC2 instance's role
```

**2. S3 Bucket Misconfiguration**

Public S3 buckets have exposed everything from government data to private photos:

```bash
# Enumerate buckets for a target (try variations of company name)
aws s3 ls s3://company-name --no-sign-request  # No credentials needed
aws s3 ls s3://company-name-backup --no-sign-request
aws s3 ls s3://company-name-dev --no-sign-request

# Download everything from a public bucket
aws s3 sync s3://bucket-name . --no-sign-request

# Check bucket ACL and policy
aws s3api get-bucket-acl --bucket bucket-name
aws s3api get-bucket-policy --bucket bucket-name

# Tools
pip install cloud_enum   # Enumerate cloud resources across AWS, Azure, GCP
cloud_enum -k companyname
```

**3. Instance Metadata Service (IMDS) via SSRF**

If you find an SSRF vulnerability in an EC2-hosted application, hit the metadata endpoint:

```bash
# From a vulnerable SSRF:
http://169.254.169.254/latest/meta-data/

# Useful paths:
http://169.254.169.254/latest/meta-data/iam/security-credentials/
# → Returns role name attached to this instance

http://169.254.169.254/latest/meta-data/iam/security-credentials/ROLE-NAME
# → Returns: AccessKeyId, SecretAccessKey, Token (temporary credentials!)

# With those credentials:
export AWS_ACCESS_KEY_ID=ASIA...
export AWS_SECRET_ACCESS_KEY=...
export AWS_SESSION_TOKEN=...
aws sts get-caller-identity  # Confirm who you are
```

**Mitigation: IMDSv2** (requires session token, blocks simple SSRF):

```bash
# Check if instance uses IMDSv2
aws ec2 describe-instances --query 'Reservations[*].Instances[*].MetadataOptions'
# Look for: HttpTokens: required (IMDSv2) vs optional (IMDSv1 - vulnerable)
```

**4. IAM Privilege Escalation**

A user with limited permissions can often escalate to admin by:

```bash
# Common PrivEsc paths:
# - iam:CreatePolicyVersion → create a new version of a policy with admin permissions
# - iam:AttachUserPolicy → attach AdministratorAccess to yourself
# - iam:PassRole + ec2:RunInstances → launch EC2 with admin role, steal credentials
# - lambda:CreateFunction + iam:PassRole → create Lambda with admin role, invoke it

# Tools: Enumerate privilege escalation paths
pip install principalmapper
pmapper graph --create  # Build IAM graph
pmapper analysis        # Find PrivEsc paths
```

### Tools

| Tool | Purpose |
| :--- | :--- |
| [Pacu](https://github.com/RhinoSecurityLabs/pacu) | AWS exploitation framework |
| [ScoutSuite](https://github.com/nccgroup/ScoutSuite) | Multi-cloud security auditing |
| [CloudMapper](https://github.com/duo-labs/cloudmapper) | AWS account visualization |
| [PMapper](https://github.com/nccgroup/PMapper) | IAM privilege escalation analysis |
| [Prowler](https://github.com/prowler-cloud/prowler) | AWS security best practices scanner |
| [TruffleHog](https://github.com/trufflesecurity/trufflehog) | Secret scanning (GitHub, S3, etc.) |

```bash
# ScoutSuite — comprehensive cloud security audit
pip install scoutsuite
scout aws --profile your-lab-profile --report-dir ./report

# Prowler — AWS security checks
pip install prowler
prowler aws --profile your-lab-profile
```

---

## Phase 3 — Cloud Defense

### Logging — Turn It On

```bash
# Enable CloudTrail (API logging) — most critical
aws cloudtrail create-trail --name fpszero-audit-trail \
  --s3-bucket-name your-cloudtrail-bucket \
  --is-multi-region-trail \
  --enable-log-file-validation
aws cloudtrail start-logging --name fpszero-audit-trail

# Enable GuardDuty (threat detection)
aws guardduty create-detector --enable

# VPC Flow Logs (network traffic)
aws ec2 create-flow-logs \
  --resource-type VPC \
  --resource-ids vpc-xxxxxxxx \
  --traffic-type ALL \
  --log-destination-type cloud-watch-logs \
  --log-group-name /aws/vpc/flowlogs
```

### IAM Hardening Checklist

```bash
# Check root account status
aws iam get-account-summary | grep RootAccessKeys  # Should be 0

# List users with console access and no MFA
aws iam list-users --query 'Users[*].UserName' --output text | while read user; do
  mfa=$(aws iam list-mfa-devices --user-name "$user" --query 'MFADevices' --output text)
  [ -z "$mfa" ] && echo "NO MFA: $user"
done

# Find overly permissive policies
aws iam list-policies --scope Local --query 'Policies[*].PolicyName'
# Review each with: aws iam get-policy-version ...

# Find access keys older than 90 days
aws iam generate-credential-report
aws iam get-credential-report --query 'Content' --output text | \
  python3 -c "import sys,base64,csv; [print(r) for r in csv.DictReader(base64.b64decode(sys.stdin.read()).decode().splitlines()) if r.get('access_key_1_last_rotated','N/A') < '2024-01-01']"
```

### Security Guardrails (Service Control Policies)

```json
// SCP: Prevent disabling CloudTrail
{
  "Effect": "Deny",
  "Action": [
    "cloudtrail:StopLogging",
    "cloudtrail:DeleteTrail",
    "cloudtrail:UpdateTrail"
  ],
  "Resource": "*"
}
```

---

## Phase 4 — Multi-Cloud

Once you have AWS solid, the patterns transfer:

| Concept | AWS | GCP | Azure |
| :--- | :--- | :--- | :--- |
| Identity | IAM | IAM | Azure AD / Entra ID |
| Audit logs | CloudTrail | Cloud Audit Logs | Activity Log |
| Object storage | S3 | Cloud Storage | Blob Storage |
| Compute | EC2 | Compute Engine | Azure VMs |
| Serverless | Lambda | Cloud Functions | Azure Functions |
| Metadata endpoint | 169.254.169.254 | metadata.google.internal | 169.254.169.254 |
| Threat detection | GuardDuty | Security Command Center | Microsoft Defender |

---

## Practice Labs

| Platform | Focus | Cost |
| :--- | :--- | :--- |
| [flaws.cloud](http://flaws.cloud/) | AWS misconfiguration — attacker perspective | Free |
| [flaws2.cloud](http://flaws2.cloud/) | AWS — both attacker and defender paths | Free |
| [CloudGoat](https://github.com/RhinoSecurityLabs/cloudgoat) | Vulnerable-by-design AWS scenarios | Free (your AWS account) |
| [Damn Vulnerable Cloud Application](https://github.com/m6a-UdS/dvca) | Multi-vuln cloud app | Free |
| [TryHackMe — AWS Rooms](https://tryhackme.com/r/room/awsbasics) | Guided AWS security | Free tier |
| [HackTheBox Cloud Labs](https://academy.hackthebox.com/) | Cloud pentesting | Subscription |

---

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Hacking the Cloud](https://hackingthe.cloud/) | Attack techniques wiki | Free |
| [AWS Security Best Practices](https://aws.amazon.com/security/security-learning/) | Defense | Free |
| [Rhino Security Labs Blog](https://rhinosecuritylabs.com/blog/) | Cloud attack research | Free |
| [Cloud Security Alliance](https://cloudsecurityalliance.org/) | Standards & research | Free |
| [Security Engineering on AWS (cert)](https://aws.amazon.com/certification/certified-security-specialty/) | AWS Security Specialty cert | ~$300 |
