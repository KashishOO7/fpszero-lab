# Quantum Computing

> See the [Quantum Computing Roadmap →](../../roadmaps/quantum.md) for the full learning path from prerequisites to PQC implementation.

This page is the reference synthesis — condensed technical knowledge for practitioners.

---

## Core Concepts Reference

### The Qubit

A qubit is the quantum analog of a classical bit. The key difference:

```
Classical bit:   |0⟩ or |1⟩         (definite, one or the other)
Qubit:          α|0⟩ + β|1⟩         (superposition, both simultaneously)
                where |α|² + |β|² = 1

Measurement collapses the superposition:
- Probability |α|² of getting 0
- Probability |β|² of getting 1
```

Physical implementations: superconducting circuits (IBM, Google), trapped ions (IonQ, Honeywell), photonic (PsiQuantum), topological (Microsoft).

### Quantum Gates (Quick Reference)

```python
import numpy as np

# Key single-qubit gates
I = np.eye(2)                                    # Identity
X = np.array([[0,1],[1,0]])                      # NOT gate (bit flip)
Z = np.array([[1,0],[0,-1]])                     # Phase flip
H = (1/np.sqrt(2)) * np.array([[1,1],[1,-1]])    # Hadamard (superposition)
S = np.array([[1,0],[0,1j]])                     # Phase gate (π/2)
T = np.array([[1,0],[0,np.exp(1j*np.pi/4)]])    # T gate (π/4)

# Two-qubit gate (4x4 matrix)
CNOT = np.array([[1,0,0,0],
                  [0,1,0,0],
                  [0,0,0,1],
                  [0,0,1,0]])   # Flips target if control = |1⟩
```

---

## Security-Critical Algorithms

### Shor's Algorithm

**Problem solved:** Integer factorization and discrete logarithm — the mathematical foundations of RSA and ECC.

**Classical complexity:** Sub-exponential but superpolynomial — hard enough that RSA-2048 is infeasible to break classically.

**Quantum complexity:** Polynomial — `O((log N)³)` — breaks RSA-2048 efficiently.

**Current state:** Shor's has been demonstrated on small numbers (15, 21, etc.) but requires thousands of **logical** (error-corrected) qubits. Today's machines have hundreds to thousands of *physical* qubits, but each logical qubit needs ~1,000 physical qubits for error correction. We are approximately 10,000× below the qubit count needed to break RSA-2048.

```
RSA-2048 vulnerability estimate:
- Logical qubits needed: ~4,000
- Physical qubits needed (with error correction): ~4,000,000
- Best current machines: ~1,000–2,000 physical qubits (noisy)
- Estimated year of cryptographic relevance: 2030–2040+ (varies widely by expert)
```

### Grover's Algorithm

**Problem solved:** Unstructured database search — find one item in N unsorted items.

**Classical complexity:** O(N) — check everything
**Quantum complexity:** O(√N) — quadratic speedup

**Security impact (concrete):**

| Algorithm | Classical Security | After Grover | Action |
| :--- | :--- | :--- | :--- |
| AES-128 | 128-bit | **64-bit** (broken) | Upgrade to AES-256 |
| AES-256 | 256-bit | 128-bit (acceptable) | Keep, you're fine |
| SHA-256 | 256-bit | 128-bit (acceptable) | Use SHA-512 for signatures |
| SHA-3-256 | 256-bit | 128-bit (acceptable) | Keep |

---

## Post-Quantum Cryptography (PQC)

### NIST Standards (August 2024)

NIST finalized four PQC standards. These are the official replacements:

```
ML-KEM (FIPS 203)     — CRYSTALS-Kyber
  Purpose: Key Encapsulation Mechanism (replaces RSA/ECDH for key exchange)
  Security basis: Learning With Errors (LWE) problem (lattice-based)
  
ML-DSA (FIPS 204)     — CRYSTALS-Dilithium  
  Purpose: Digital Signatures
  Security basis: Module LWE / Module SIS (lattice-based)
  
SLH-DSA (FIPS 205)    — SPHINCS+
  Purpose: Digital Signatures (hash-based fallback)
  Security basis: Hash function security only (conservative choice)
  
FN-DSA (FIPS 206)     — FALCON
  Purpose: Digital Signatures (compact)
  Security basis: NTRU lattice problem
```

### Implementation Example

```python
# pip install liboqs-python
import oqs

# Key Exchange with ML-KEM (Kyber)
print("=== ML-KEM Key Exchange ===")
kem = oqs.KeyEncapsulation('Kyber768')

# Recipient generates keypair
public_key = kem.generate_keypair()
secret_key = kem.export_secret_key()

# Sender encapsulates
ciphertext, shared_secret_sender = kem.encap_secret(public_key)
print(f"Shared secret (sender):  {shared_secret_sender.hex()[:32]}...")

# Recipient decapsulates
kem2 = oqs.KeyEncapsulation('Kyber768', secret_key)
shared_secret_recipient = kem2.decap_secret(ciphertext)
print(f"Shared secret (recip.):  {shared_secret_recipient.hex()[:32]}...")

assert shared_secret_sender == shared_secret_recipient
print("✓ Shared secrets match — secure channel established")

# Digital Signature with ML-DSA (Dilithium)
print("\n=== ML-DSA Signature ===")
sig = oqs.Signature('Dilithium3')
public_key = sig.generate_keypair()

message = b"fpszero lab — authenticated message"
signature = sig.sign(message)

verifier = oqs.Signature('Dilithium3')
is_valid = verifier.verify(message, signature, public_key)
print(f"Signature valid: {is_valid}")
```

### Migration Checklist

For any system using public-key cryptography:

```
□ Inventory all asymmetric cryptography in use
  - TLS certificates and versions
  - SSH keys (RSA, ECDSA, Ed25519)
  - Code signing keys
  - Encryption at rest schemes
  - API authentication tokens (JWT with RS256/ES256)

□ Prioritize by data sensitivity and lifetime
  - Data that must stay secret for 20+ years: migrate NOW
  - Short-lived session keys: can wait but plan now
  
□ Enable "hybrid" mode first (classical + PQC in parallel)
  - TLS 1.3 + X25519Kyber768 (Google/Cloudflare have deployed this)
  
□ Test PQC performance
  - Kyber keys are larger than RSA keys → check handshake size budgets
  - Dilithium signatures are larger than ECDSA → check signature size constraints
```

---

## Quantum Computing Resources

| Resource | Link | Type |
| :--- | :--- | :--- |
| IBM Quantum (free hardware access) | [quantum.ibm.com](https://quantum.ibm.com) | Platform |
| Qiskit Textbook | [github.com/Qiskit/textbook](https://github.com/Qiskit/textbook) | Free book |
| NIST PQC Standards | [csrc.nist.gov/pqc](https://csrc.nist.gov/projects/post-quantum-cryptography) | Standards |
| Open Quantum Safe (liboqs) | [openquantumsafe.org](https://openquantumsafe.org) | Implementation |
| Quantum Country (essays) | [quantum.country](https://quantum.country) | Reading |
