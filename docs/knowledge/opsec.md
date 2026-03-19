# OPSEC — Operational Security

> The best defense is not being seen. OPSEC is the discipline of denying adversaries the information they need to act against you.

Operational Security is the process of identifying, controlling, and protecting information that could be used against you. It applies to security researchers, journalists, activists, businesses, and anyone operating in an adversarial environment.

---

## The OPSEC Process

Originally a military concept (OPSEC was formalized during the Vietnam War as "Purple Dragon"), the process has five steps:

1. **Identify critical information** — What, if exposed, would hurt you?
2. **Analyze threats** — Who is your adversary? What are their capabilities?
3. **Analyze vulnerabilities** — Where are you leaking that critical information?
4. **Assess risk** — Which vulnerabilities are most likely to be exploited?
5. **Apply countermeasures** — Reduce exposure at acceptable cost

!!! abstract "fps advice"
    OPSEC is threat-model-dependent. A journalist protecting a source has different requirements than a pentester protecting client data. Define your adversary first — everything else follows from that.

---

## Practical OPSEC Principles

**Identity separation:** Keep professional and personal identities isolated. Separate email addresses, browsers, devices, or at minimum browser profiles.

**Metadata awareness:** Every file, photo, and document carries metadata — timestamps, GPS coordinates, author names, software versions. Strip it before sharing.

**Communication security:** Use end-to-end encrypted messaging (Signal) for sensitive communication. Understand that metadata (who contacted whom, when) can be as revealing as content.

**Infrastructure hygiene:** Use VPNs, Tor, or both depending on threat model. Understand what each does and does not protect against. A VPN shifts trust from your ISP to the VPN provider — choose accordingly.

**Digital footprint management:** Audit what is publicly visible about you. Use a custom DNS resolver. Minimize browser fingerprint surface. Disable telemetry where possible.

---

## Resources

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [Operations Security (Wikipedia)](https://en.wikipedia.org/wiki/Operations_security) | Reference | OPSEC process overview with military origins |
| [EFF Surveillance Self-Defense](https://ssd.eff.org/) | Guide | Practical digital privacy and OPSEC |
| [Privacy Guides](https://www.privacyguides.org/) | Guide | Community-maintained privacy tool recommendations |
| [Tails OS](https://tails.net/) | Tool | Amnesic live OS for high-OPSEC scenarios |
| [Whonix](https://www.whonix.org/) | Tool | OS designed for anonymity via Tor |

---

## Landscape

- Threat modeling frameworks (STRIDE, PASTA)
- OPSEC for bug bounty hunters and security researchers
- Counter-OSINT: minimizing your own digital footprint
- Operational security for open source contributors

---

!!! abstract "fps advice"
    Perfect OPSEC is impossible. The goal is to make the cost of attacking you higher than the value of what you're protecting. Know your threat model. Act accordingly.