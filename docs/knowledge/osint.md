# OSINT — Open Source Intelligence

> The best intelligence doesn't require breaking in. It requires knowing where to look.

OSINT is the practice of collecting and analyzing information from publicly available sources — search engines, social media, public records, DNS, WHOIS, satellite imagery, government filings, and more.

---

## Why OSINT Matters

Every penetration test starts with reconnaissance. Every investigation starts with what's publicly visible. OSINT is the foundation of both offense and defense:

- **Red team:** Map the target's attack surface before touching a single system
- **Blue team:** Understand what an adversary can see about your organization
- **Investigations:** Trace identities, infrastructure, and connections across open data

---

## The OSINT Mindset

!!! abstract "fps advice"
    The less you have, the better you are. OSINT is not about hoarding data — it's about extracting signal from noise. One verified connection is worth more than a thousand unfiltered search results.

The skill is not in the tools. The skill is in knowing what question to ask, where the answer might live, and how to verify it without contaminating the source.

---

## Core Domains

| Domain | What You're Looking For |
| :--- | :--- |
| **People** | Identities, social media, email, phone, employment history |
| **Infrastructure** | Domains, IPs, ASNs, DNS records, certificate transparency |
| **Organizations** | Corporate filings, patents, job postings (reveal tech stack), partnerships |
| **Geolocation** | Satellite imagery, street-level data, metadata from photos |
| **Dark web** | Paste sites, forums, marketplaces (observe, do not participate) |
| **Code** | GitHub repos, leaked credentials, API keys in public repos |

---

## Essential Resources

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [awesome-osint](https://github.com/jivoi/awesome-osint) | Curated list | The definitive OSINT resource collection on GitHub |
| [OSINT Framework](https://osintframework.com/) | Visual tool map | Categorized tree of OSINT tools by purpose |
| [IntelTechniques](https://inteltechniques.com/tools/) | Tool collection | Michael Bazzell's OSINT tools and methodology |
| [Shodan](https://shodan.io) | Search engine | Internet-wide device and service discovery |
| [theHarvester](https://github.com/laramies/theHarvester) | Tool | Emails, subdomains, IPs from public sources |
| [SpiderFoot](https://github.com/smicallef/spiderfoot) | Tool | Automated OSINT collection and correlation |
| [Maltego](https://maltego.com) | Tool | Link analysis and visual intelligence mapping |
| [crt.sh](https://crt.sh/) | Tool | Certificate transparency log search |
| [Wayback Machine](https://web.archive.org/) | Archive | Historical snapshots of websites |

---

## Landscape

These are directions for deeper OSINT coverage — not current content:

- Geospatial OSINT (GEOINT)
- Social media analysis and sock puppet methodology
- Corporate intelligence and financial OSINT
- Dark web monitoring (ethical, observation-only)
- OSINT automation pipelines

---

!!! warning "Ethics"
    OSINT uses publicly available information only. Do not access private systems, circumvent access controls, or harass individuals. The line between research and stalking is real. Stay on the right side of it.
