# Knowledge Synthesis

> The gap between reading about security and *doing* security is closed by deeply understanding fundamentals, not by memorizing tool flags.

This section breaks down complex systems to their base logic. No vendor buzzwords. No certification-dump summaries. Architecture, reasoning, and first principles.

---

## The Knowledge Map

### Systems — The Foundation Layer

Everything runs on something. Before you can attack or defend, you need to understand what you're dealing with at the hardware and OS level.

- [Architecture](systems.md) — How digital systems are structured
- [Linux](systems/linux.md) — Kernel, shell, internals
- [Networking](systems/networking.md) — TCP/IP, protocols, packet flow

### Offense — Red Team

Understanding attack techniques is not about hacking — it's about knowing how systems fail so you can make them not fail.

- [Overview](offense.md) — Mindset and methodology
- [Web Security](offense/web_sec.md) — OWASP, injection, logic flaws
- [Malware](offense/malware.md) — Persistence, C2, analysis
- [Active Directory](offense/active_directory.md) — Enterprise attack paths

### Defense — Blue Team

Detection is harder than attack. Building detection that works requires understanding what attacker behavior *looks like* in logs and on the wire.

- [Operations](defense.md) — Blue team strategy, SIEM, monitoring
- [Forensics](defense/forensics.md) — Digital forensics and incident response

### Deep Tech — The Frontier

Domains where the underlying physics or math is non-obvious and the security implications are underappreciated.

- [Frontier Overview](deep_tech.md)
- [Quantum Computing](deep_tech/quantum.md) — Qubits, algorithms, PQC
- [Blockchain](deep_tech/blockchain.md) — Trustless systems, smart contract flaws

### Mobile

- [Mobile Security](mobile.md) — Android and iOS — architecture, pentesting, forensics

### OSINT

- [Open Source Intelligence](osint.md) — Reconnaissance, infrastructure mapping, identity research

### OPSEC

- [Operational Security](opsec.md) — Information control, threat modeling, digital footprint management

### AI/ML Security

- [AI/ML Security](ai_ml_security.md) — Adversarial attacks, model integrity, prompt injection, supply chain

---

!!! note "Living Document"
    These pages are updated as understanding evolves. Dated information will be flagged.
