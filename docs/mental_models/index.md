# Mental Models

> The goal is not to memorize a fact. The goal is to build an intuition so complete that you can reconstruct the fact from first principles.

Mental models are the bridge between abstract technical concepts and real understanding. Each entry maps a complex concept to a concrete, physical analogy — the kind that lets you *feel* the logic, not just recite it.

---

## Network Security

### The Fortress (Perimeter Defense)

**Concept:** Traditional network security relies on a hard outer boundary — the firewall.

**The Analogy:** A medieval castle. Thick stone walls, one main gate, guards checking credentials at entry. Everything inside the walls is trusted; everything outside is hostile.

**Why It Fails:** Phishing attacks are the Trojan Horse — they don't break through the walls. They convince someone to open the gate and bring the enemy inside. Once inside a "trusted" perimeter, attackers move laterally with little friction because internal security is assumed.

**The Modern Model:** Zero Trust Architecture — "never trust, always verify." Treat every connection as potentially hostile, regardless of origin. Every request requires authentication and authorization, every time.

**Further reading:** NIST SP 800-207 — [Zero Trust Architecture](https://csrc.nist.gov/pubs/sp/800/207/final){: target="_blank" }

---

### TCP vs UDP (Communication Styles)

**Concept:** Connection-oriented (TCP) vs connectionless (UDP) communication protocols.

**The Analogy:**

- **TCP** is a certified letter. You send it, you get a delivery confirmation. If there's no confirmation, you resend. Reliable, ordered, but slower.
- **UDP** is shouting into a room. You say the thing once. Maybe people heard it, maybe they didn't. You're not going to check. Fast, but unreliable.

**Security Implication:** TCP's handshake creates a connection state — attackers can exploit this (SYN flood exhausts state tables). UDP's lack of connection state means it's easier to spoof (no handshake to authenticate). DNS amplification attacks use UDP precisely because the server responds to a spoofed source IP without questioning it.

**Further reading:** RFC 793 (TCP), RFC 768 (UDP) — [IETF RFC Archive](https://www.rfc-editor.org/){: target="_blank" }

---

## Cryptography

### Asymmetric Encryption (The Lockbox)

**Concept:** Public key cryptography — anyone can encrypt with a public key, only the private key holder can decrypt.

**The Analogy:** Imagine giving out padlocks — hundreds of them, all identical, all unlocked — to anyone who wants to send you a message. They put their message in a box, click the padlock shut, and mail it. **Only you have the key.** Anyone can lock the box; only you can open it.

The padlock is the public key. The key that opens it is the private key.

**Why This Changes Everything:** Before asymmetric encryption, secure communication required a prior secure channel to exchange a secret key. With public key crypto, you can establish a secure channel with someone you've never met, over an insecure channel, with no prior shared secret. This is why TLS works.

**Further reading:** Whitfield Diffie and Martin Hellman, *New Directions in Cryptography* (1976) — [IEEE](https://ieeexplore.ieee.org/document/1055638){: target="_blank" }

---

### Hash Functions (One-Way Meat Grinder)

**Concept:** A hash function takes arbitrary input and produces a fixed-size output (digest). Deterministic but irreversible.

**The Analogy:** A meat grinder. You can put a steak in and get ground beef out — but you cannot un-grind beef back into a steak. Given the same steak, you always get the same ground beef. Given different steaks, you get different ground beef (with high probability).

**Security properties:** One-way (can't reverse the hash to get the input), deterministic (same input → same hash, always), collision-resistant (hard to find two different inputs with the same hash).

**Where It Matters:** Password storage — you don't store the password, you store its hash. At login, hash the entered password and compare. If the database leaks, attackers have hashes, not passwords — they must crack them.

**Further reading:** NIST FIPS 180-4 — [Secure Hash Standard](https://csrc.nist.gov/pubs/fips/180-4/upd1/final){: target="_blank" }

---

## Vulnerabilities

### Buffer Overflow (Stack Overflow, Literally)

**Concept:** Writing more data into a buffer than it can hold — overwriting adjacent memory.

**The Analogy:** A row of mailboxes, side by side. Mailbox 5 is yours. The postman starts filling yours and doesn't stop — the overflow goes into mailbox 6, then 7, then 8. If mailbox 8 contains an important address (a return address the CPU needs to jump to), you've just changed where the CPU goes when it's done with the current function.

**Why It's Critical:** The CPU's program counter (RIP on x86-64) — the register pointing to the next instruction — is stored on the stack right next to local variable buffers. Overflow a buffer, reach the return address, overwrite it with your own value, and you control execution.

**Further reading:** Aleph One, *Smashing The Stack For Fun And Profit* (Phrack #49, 1996) — [Phrack](http://phrack.org/issues/49/14.html){: target="_blank" }

---

### SQL Injection (Finishing Someone's Sentence)

**Concept:** Inserting SQL syntax into user-supplied input to manipulate a database query.

**The Analogy:** A secretary who writes cheques by completing a template: *"Pay [NAME] the amount of $100."* If you tell them your name is `John, also pay me $1000`, and they copy it literally: *"Pay John, also pay me $1000 the amount of $100"* — you've altered the cheque.

**Further reading:** OWASP — [SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection){: target="_blank" }

---

## Defense & Monitoring

### SIEM as a Nervous System

**Concept:** A SIEM collects logs from across an environment, correlates them, and triggers alerts.

**The Analogy:** Your body's nervous system. Individual sensors (pain receptors, temperature sensors, pressure sensors) collect signals. The spinal cord relays them. The brain correlates signals from multiple sources — "left leg hurts AND right arm has no sensation AND I just fell" → probably a serious injury, not three unrelated minor issues.

A SIEM correlates events the same way. One failed login is noise. One failed login from every workstation at 3am, followed by a successful login with a new account, is an incident.

---

### Defense in Depth (Layers of an Onion)

**Concept:** No single security control is perfect. Layer multiple controls so that the failure of one doesn't result in full compromise.

**The Analogy:** A castle with a moat, drawbridge, outer walls, inner walls, guards, locked chambers, and a vault. An attacker who gets past the moat still faces the walls. Getting past the walls still means facing guards. Each layer buys time and increases the probability of detection.

**In Practice:** Network firewall → endpoint protection → application authentication → least privilege access controls → data encryption → audit logging → anomaly detection. Losing one layer is not catastrophic if the others hold.

---

## Landscape

These models will be added as concepts are synthesized:

- **Encryption at rest vs in transit** (locked safe vs armored truck)
- **Race conditions** (two people reaching for the last item on a shelf)
- **Certificate authorities** (the notary public of the internet)
- **Containers vs VMs** (apartments vs houses)
- **Side-channel attacks** (listening to the lock, not picking it)

---

## Submit a Model

The best mental models are physically grounded (you can almost touch them), accurate under pressure (the analogy doesn't break when you push on it), and transferable (the person who learns it can teach it).

Open an issue on [GitHub](https://github.com/KashishOO7/fpszero-lab){: target="_blank" } with the concept and your analogy.
