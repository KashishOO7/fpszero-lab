# Bug Bounty Roadmap

**Goal:** Find real security vulnerabilities in production applications, report them responsibly, and get paid for it. Build the skills, the process, and the patience to do it consistently.

!!! warning "Scope is Law"
    Bug bounty is **not** a license to hack anything. Every program has a defined scope.
    Testing out-of-scope assets is illegal, even if you find a vulnerability.
    Read the program policy before you touch anything. When in doubt, ask the program.

---

## What Bug Bounty Actually Is

Bug bounty is applied offensive security at scale. Unlike CTFs (artificial challenges) or lab pentests (controlled environments), bug bounty is:

- **Real targets** — production systems used by real people
- **Real consequences** — vulnerabilities can affect millions of users
- **No walkthroughs** — you figure out what's broken without hints
- **Variable payouts** — critical = thousands of dollars, informational = nothing
- **Competitive** — other researchers are looking at the same targets

The truth most people don't say: **most beginners earn nothing for the first 6–12 months.** Bug bounty is not passive income. It rewards depth of knowledge and systematic methodology.

---

## Phase 0 — Prerequisites (Non-Negotiable)

Do not start bug bounty until you can answer yes to all of these:

- [ ] I can explain how HTTP, cookies, and sessions work at the protocol level
- [ ] I can set up and use Burp Suite to intercept and modify requests
- [ ] I can explain SQL injection and demonstrate it on a lab (DVWA, PortSwigger)
- [ ] I can explain XSS (reflected, stored, DOM) and write payloads manually
- [ ] I have completed at least 10 PortSwigger Web Security Academy labs
- [ ] I understand what IDOR is and have found one in a practice environment
- [ ] I can do basic recon — subdomain enumeration, directory bruteforcing

If you're not there yet: [Web Security Knowledge →](../knowledge/offense/web_sec.md) then come back.

---

## Phase 1 — The Methodology

Methodology beats tools. A systematic process finds bugs that automated scanners miss.

### The Bug Bounty Workflow

```
1. TARGET SELECTION
   ↓
2. RECON (wide)
   ↓
3. ATTACK SURFACE MAPPING
   ↓
4. VULNERABILITY HUNTING (deep)
   ↓
5. EXPLOITATION & PROOF OF CONCEPT
   ↓
6. REPORT WRITING
   ↓
7. TRIAGE & DISCLOSURE
```

### Step 1: Target Selection

**For beginners:** Start with programs that have a large scope and are known to be responsive.

```
Good programs to start on:
- HackerOne: HackerOne itself (https://hackerone.com/security)
- Bugcrowd: Bugcrowd itself
- Programs with "VDP" (Vulnerability Disclosure Program) — no payout, but good for learning
- Programs with large scope (*.example.com vs just www.example.com)

Avoid initially:
- Programs with tiny scope (one login page)
- Programs marked "low signal" (everything is a duplicate)
- Programs with very long triage times (discouraging for learning)
```

**Finding targets:**
- [HackerOne](https://hackerone.com/bug-bounty-programs)
- [Bugcrowd](https://bugcrowd.com/programs)
- [Intigriti](https://www.intigriti.com/)
- [Synack](https://www.synack.com/) (invitation-only, higher signal)
- [YesWeHack](https://www.yeswehack.com/)

### Step 2: Recon — Go Wide Before You Go Deep

The goal of recon is to find the full attack surface — everything the company exposes to the internet.

```bash
# Illustrative workflow — specific flags and tools evolve, always check current docs
# === SUBDOMAIN ENUMERATION ===
# Passive (no direct contact with target)
subfinder -d target.com -o subdomains.txt
amass enum -passive -d target.com -o amass.txt
assetfinder --subs-only target.com

# DNS brute force (active)
gobuster dns -d target.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt

# Certificate transparency (find subdomains from SSL certs)
curl -s "https://crt.sh/?q=%.target.com&output=json" | \
  python3 -c "import json,sys; [print(x['name_value']) for x in json.load(sys.stdin)]" | \
  sort -u

# Combine and deduplicate
cat subdomains.txt amass.txt | sort -u > all_subdomains.txt

# === CHECK WHAT'S ALIVE ===
cat all_subdomains.txt | httpx -silent -status-code -title -o live_hosts.txt

# === SCREENSHOT ALL LIVE HOSTS ===
# (Visual recon — spot interesting apps fast)
gowitness file -f live_hosts.txt --screenshot-path ./screens/

# === PORT SCANNING (only if in scope) ===
nmap -iL live_hosts.txt -sV --open -oN ports.txt
```

**What to look for in recon:**
- Staging/dev/test subdomains (often less hardened than prod)
- Old or forgotten applications
- Admin panels
- API endpoints (api.target.com, api-v2.target.com)
- Different tech stacks (the main site is hardened; that old PHP app is not)

### Step 3: Attack Surface Mapping

For each live host:

```bash
# Technology fingerprinting
whatweb https://target.com
wappalyzer (browser extension)
curl -I https://target.com | grep -i "server\|x-powered-by\|set-cookie"

# Directory enumeration
ffuf -u https://target.com/FUZZ \
  -w /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt \
  -mc 200,201,301,302,401,403 -o dirs.txt

# JavaScript file analysis (secrets, API endpoints, hidden parameters)
# Tools: getJS, LinkFinder, SecretFinder
getJS --url https://target.com --output jsfiles.txt
cat jsfiles.txt | while read url; do
  curl -s "$url" | grep -oE '(api|/v[0-9]|/graphql)[^"]*'
done

# Parameter discovery
arjun -u https://target.com/api/endpoint -oT params.txt
```

### Step 4: Vulnerability Hunting

**The mindset shift:** Don't look for "vulnerabilities." Look for **functionality that can be abused.**

Every feature is a potential vulnerability. Ask about every function:
- *What does this do?*
- *What does the developer assume about the input?*
- *What happens if those assumptions are wrong?*

**High-value targets by impact:**

| Vulnerability Class | Typical Payout | Difficulty |
| :--- | :--- | :--- |
| Remote Code Execution (RCE) | $5k–$50k+ | High |
| Authentication bypass | $2k–$20k | Medium |
| SQL Injection | $1k–$10k | Medium |
| SSRF (internal access) | $1k–$10k | Medium |
| IDOR (sensitive data) | $500–$5k | Low–Medium |
| XSS (account takeover) | $500–$3k | Low–Medium |
| Sensitive data exposure | $500–$5k | Low |
| CSRF | $200–$1k | Low |
| Open Redirect | $50–$300 | Low |
| Missing rate limiting | Often N/A | Very Low |

**Business logic bugs** often pay the most and are hardest to find with automation:
- Can I buy something for $0?
- Can I access another user's data by changing an ID?
- Can I skip a required step in a workflow?
- Can I apply a coupon code unlimited times?
- What happens when I manipulate the price parameter?

---

## Phase 2 — Burp Suite Mastery

Burp Suite Community (free) is your primary tool. Learn it deeply.

```
Core workflows:
1. Proxy → Intercept → modify any request on the fly
2. Repeater → send the same modified request repeatedly (manual testing)
3. Intruder → automated fuzzing with payloads
4. Decoder → encode/decode payloads (Base64, URL, HTML, etc.)
5. Comparer → diff two responses to spot differences
```

```
Essential Burp extensions (install via BApp Store):
- Param Miner — discover hidden parameters
- Autorize — test for IDOR/access control flaws automatically
- Turbo Intruder — fast HTTP fuzzer (replaces Intruder for speed)
- JWT Editor — test JWT vulnerabilities
- Active Scan++ — additional scan checks
- Retire.js — detect outdated JS libraries
```

**The Autorize workflow for IDOR testing:**

```
1. Log in as User A
2. Configure Autorize with User B's session cookie
3. Browse the application as User A
4. Autorize replays every request as User B
5. Red rows = User B got same response = IDOR
```

---

## Phase 3 — Writing Reports

A good vulnerability report is worth more than finding the bug. A bad report gets triaged as "informational" or marked duplicate.

### Report Structure

```markdown
## Title
[Vulnerability Type] in [Location] allows [Impact]
Example: "Stored XSS in user profile bio field allows account takeover"

## Summary
One paragraph. What's vulnerable, how it's exploited, what the impact is.
Write it so a non-technical product manager understands the risk in 30 seconds.

## Steps to Reproduce
Numbered, copy-pasteable steps. Assume the reader has no context.
Include:
1. Environment (browser, OS, account type)
2. Every click, every request
3. The exact payload used
4. What the expected behavior should be
5. What the actual behavior is

## Proof of Concept
- Screenshot or screen recording showing the vulnerability
- For XSS: screenshot of alert/cookie theft
- For IDOR: show data from another account
- For RCE: `id` command output, NOT a reverse shell (never actually exploit production)

## Impact
Be specific. What can an attacker do?
Bad: "This could lead to security issues"
Good: "An attacker can steal session cookies of any user who views the affected profile,
       then use those cookies to log in as that user, accessing their private messages,
       payment history, and billing information."

## Remediation
What should the developer do to fix it? Be specific.
- "Sanitize user input using htmlspecialchars() before rendering"
- "Validate that the user_id parameter matches the authenticated user's session"
```

### Severity Ratings (CVSS approximations)

| Severity | CVSS | Examples |
| :--- | :--- | :--- |
| Critical | 9.0–10.0 | Unauthenticated RCE, auth bypass for all accounts |
| High | 7.0–8.9 | Authenticated RCE, SSRF to internal network, SQLi |
| Medium | 4.0–6.9 | XSS with account takeover, IDOR sensitive data |
| Low | 1.0–3.9 | Self-XSS, clickjacking, open redirect |
| Informational | 0 | Missing security headers, version disclosure |

---

## Phase 4 — Leveling Up

### Specialize

The most successful researchers pick a niche:

- **API Security** — REST, GraphQL, gRPC have distinct vulnerability classes
- **Mobile Bug Bounty** — Android/iOS apps (combine with Mobile Roadmap)
- **Cloud Attack Surface** — S3, Lambda, cloud-hosted apps (combine with Cloud Roadmap)
- **Authentication** — OAuth, SAML, JWT, SSO flows
- **File Upload/Processing** — PDF, image, document processing pipelines

### Read Other People's Reports

Every disclosed report is a free lesson:

- [HackerOne Hacktivity](https://hackerone.com/hacktivity) — public disclosed reports
- [Bug Bounty Reports Explained (YouTube)](https://www.youtube.com/c/BugBountyReportsExplained)
- [Pentester Land write-ups](https://pentester.land/writeups/)
- [0xpatrik blog](https://0xpatrik.com/) — subdomain takeover and recon

### Track Your Work

```
Notion / Obsidian template for each target:
- Date started
- Scope definition
- Recon notes (subdomains, technologies, interesting endpoints)
- Tested parameters and results
- Submitted reports and status
- Lessons learned
```

---

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Bug Bounty Bootcamp (Vickie Li)](https://nostarch.com/bug-bounty-bootcamp) | Book | Paid |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security) | Labs | Free |
| [NahamCon CTF](https://www.nahamcon.com/) | Competition | Free |
| [Hacker101](https://www.hacker101.com/) | Course + CTF | Free |
| [Jason Haddix's Bug Hunter's Methodology](https://www.youtube.com/watch?v=uKWu6yhnhbQ) | Video | Free |
| [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/) | Reference | Free |
| [KingOfBugBountyTips](https://github.com/KingOfBugbounty/KingOfBugBountyTips) | Tips collection | Free |
| [Stanford CS253 — Web Security](https://web.stanford.edu/class/cs253/) | Course | Free |

---

!!! tip "The Most Important Thing"
    The difference between a researcher who earns from bug bounty and one who doesn't is not knowledge of more vulnerabilities. It's **thoroughness on fewer targets**. Pick one program. Map every endpoint. Test every parameter. Find the thing that automated scanners miss because it requires understanding the application's business logic. That's where the bugs are.
