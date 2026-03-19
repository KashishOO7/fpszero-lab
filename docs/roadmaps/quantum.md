# Quantum Computing Roadmap

**Goal:** Understand quantum computing from first principles — the physics, the math, and the security implications — without needing a physics PhD.

!!! info "Why Security People Should Care"
    Quantum computers running Shor's Algorithm can theoretically break RSA and ECC — the encryption protecting most of the internet. **Post-Quantum Cryptography (PQC)** is being standardized now. Understanding *why* matters as much as understanding what to replace it with.

---

## The Honest Reality Check

Quantum computing is surrounded by hype. Let's be precise:

- **Current state (2024-2025):** Quantum computers are real but *noisy* (NISQ era — Noisy Intermediate-Scale Quantum). They can beat classical computers at specific narrow tasks (quantum supremacy demonstrations), but cannot yet break RSA or run Shor's at scale.
- **Timeline to cryptographically relevant quantum computers:** Estimates range from 10 to 30+ years. The variance is large.
- **What's certain:** Governments and large organizations are beginning PQC migration *now* because "harvest now, decrypt later" attacks are real — adversaries collect encrypted data today to decrypt it once they have the hardware.

---

## Phase 0 — Mathematical Prerequisites

Quantum computing is fundamentally linear algebra applied to probability.

### Linear Algebra (Essential)

You need to be comfortable with:

- **Vectors** — column vectors, row vectors, dot product
- **Matrices** — multiplication, transpose, inverse
- **Complex numbers** — `a + bi`, magnitude, complex conjugate
- **Eigenvectors & eigenvalues** — used in quantum gates
- **Tensor product (⊗)** — used to combine qubit states

```python
# Python: linear algebra with NumPy
import numpy as np

# A qubit |0⟩ state as a vector
zero = np.array([1, 0])
one  = np.array([0, 1])

# Hadamard gate (puts qubit in superposition)
H = (1/np.sqrt(2)) * np.array([[1, 1],
                                 [1,-1]])

# Apply H to |0⟩ → equal superposition of |0⟩ and |1⟩
print(H @ zero)   # → [0.707, 0.707]
```

**Resources:**

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [3Blue1Brown — Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) | Video | Free |
| [Khan Academy Linear Algebra](https://www.khanacademy.org/math/linear-algebra) | Interactive | Free |
| [Gilbert Strang MIT OCW](https://ocw.mit.edu/courses/18-06-linear-algebra-spring-2010/) | Full course | Free |

---

## Phase 1 — Quantum Mechanics Intuition

You don't need to solve Schrödinger's equation. You need the *concepts.*

### The Key Ideas

**1. Superposition**

A classical bit is 0 or 1. A qubit can be in a *superposition* — a weighted combination of both states simultaneously. Until you measure it, it's neither. The moment you measure, it collapses to one definite value.

Think of it like a coin spinning in the air — it's not heads or tails until it lands.

**2. Entanglement**

Two qubits can be *entangled* so that measuring one instantly determines the state of the other, regardless of distance. Einstein called this "spooky action at a distance."

Entanglement is a resource — it lets quantum computers process correlated information that classical computers cannot.

**3. Interference**

Quantum algorithms exploit interference: paths that lead to wrong answers cancel out (destructive interference), paths that lead to correct answers reinforce (constructive interference). This is how quantum algorithms achieve speedup.

### Dirac Notation (Bra-Ket)

The standard notation you'll see everywhere:

| Symbol | Meaning |
| :--- | :--- |
| `\|0⟩` | Qubit in state zero (column vector [1,0]) |
| `\|1⟩` | Qubit in state one (column vector [0,1]) |
| `\|ψ⟩` | Arbitrary quantum state |
| `α\|0⟩ + β\|1⟩` | Superposition (α,β are complex amplitudes; `\|α\|²+\|β\|²=1`) |
| `⟨ψ\|` | Conjugate transpose of `\|ψ⟩` |
| `⟨0\|1⟩` | Inner product (= 0, states are orthogonal) |

---

## Phase 2 — Quantum Computing

### Quantum Gates

Like classical logic gates (AND, OR, NOT), quantum gates are *unitary matrices* that transform qubit states. All quantum gates are **reversible.**

| Gate | Symbol | Operation |
| :--- | :--- | :--- |
| Pauli-X (NOT) | X | Flips `\|0⟩↔\|1⟩` |
| Hadamard | H | Creates superposition from `\|0⟩` or `\|1⟩` |
| Pauli-Z | Z | Phase flip (changes sign of `\|1⟩`) |
| CNOT | CX | Controlled-NOT: flips target if control = `\|1⟩` |
| Toffoli | CCX | Controlled-Controlled-NOT |

```python
# Illustrative example — Qiskit API may evolve, verify against current docs
# pip install qiskit qiskit-aer

from qiskit import QuantumCircuit
from qiskit_aer import AerSimulator

# Build a Bell state (maximally entangled pair of qubits)
qc = QuantumCircuit(2, 2)
qc.h(0)          # Hadamard on qubit 0 → superposition
qc.cx(0, 1)      # CNOT with qubit 0 as control → entanglement
qc.measure([0,1], [0,1])

# Simulate
sim = AerSimulator()
result = sim.run(qc, shots=1000).result()
print(result.get_counts())  
# → {'00': ~500, '11': ~500} — perfectly correlated
```

### Key Quantum Algorithms

**Grover's Algorithm** (search speedup)

- Classical search: O(N) — check every item
- Grover's search: O(√N) — quadratic speedup
- Security impact: **halves effective key length.** AES-128 → equivalent to 64-bit classically. Solution: use AES-256.

**Shor's Algorithm** (factoring)

- Classical factoring: exponential time — this is the basis of RSA's security
- Shor's: polynomial time — **breaks RSA and ECC entirely** on a sufficiently powerful quantum computer
- Security impact: RSA-2048 is eventually vulnerable. Migration to PQC is necessary.

**Deutsch-Jozsa Algorithm** (proof of quantum speedup, not practically useful)

**Quantum Phase Estimation** (used in many advanced algorithms)

---

## Phase 3 — Post-Quantum Cryptography (PQC)

This is where quantum computing directly intersects with security practice.

### What Breaks and What Doesn't

| Algorithm | Vulnerable? | Why |
| :--- | :--- | :--- |
| RSA | **Yes** | Shor's factors n=p*q |
| ECC / ECDSA | **Yes** | Shor's solves discrete log problem |
| Diffie-Hellman | **Yes** | Same discrete log problem |
| AES-128 | Weakened | Grover's halves key strength → use AES-256 |
| AES-256 | **No** (effectively) | Grover's leaves 128-bit security, sufficient |
| SHA-256 | **No** | Grover's: SHA-512 preferred for signatures |
| SHA-3 | **No** | Quantum-resistant |

### NIST PQC Standards (Finalized 2024)

NIST standardized 4 post-quantum algorithms:

| Algorithm | Type | Purpose |
| :--- | :--- | :--- |
| **ML-KEM** (CRYSTALS-Kyber) | Lattice-based | Key encapsulation (replaces RSA/ECDH key exchange) |
| **ML-DSA** (CRYSTALS-Dilithium) | Lattice-based | Digital signatures |
| **SLH-DSA** (SPHINCS+) | Hash-based | Digital signatures (conservative backup) |
| **FN-DSA** (FALCON) | Lattice-based | Digital signatures (compact) |

```python
# Using liboqs-python for post-quantum crypto
# pip install liboqs-python

import oqs

# Key encapsulation with Kyber
kem = oqs.KeyEncapsulation('Kyber512')
public_key = kem.generate_keypair()

# Encapsulate (sender side)
ciphertext, shared_secret_enc = kem.encap_secret(public_key)

# Decapsulate (receiver side)
shared_secret_dec = kem.decap_secret(ciphertext)

assert shared_secret_enc == shared_secret_dec  # Same shared secret
```

---

## Phase 4 — Hands-On Practice

### Quantum Computing Platforms (All Free Tier)

| Platform | Access | Features |
| :--- | :--- | :--- |
| [IBM Quantum](https://quantum.ibm.com/) | Free account | Real quantum hardware + Qiskit |
| [Google Cirq](https://quantumai.google/cirq) | Free | Python framework + simulators |
| [Amazon Braket](https://aws.amazon.com/braket/) | Free tier | Multiple backends |
| [Quirk](https://algassert.com/quirk) | Browser-based | Visual circuit builder, no account needed |

### Exercises

1. Build a Bell state (see Phase 2 code above) and verify entanglement by running 1000 shots
2. Implement Grover's algorithm on a 2-qubit system to search a 4-element database
3. Run Deutsch-Jozsa on IBM Quantum's actual hardware (feel the noise)
4. Implement ML-KEM key exchange in Python using `liboqs`
5. Read NIST FIPS 203 (ML-KEM standard) — at least the executive summary

---

## Learning Resources

| Resource | Type | Cost |
| :--- | :--- | :--- |
| [Qiskit Textbook](https://github.com/Qiskit/textbook) | Interactive textbook | Free |
| [Quantum Country](https://quantum.country/) | Mnemonic-based essays | Free |
| [Nielsen & Chuang — Quantum Computation and Quantum Information](https://www.cambridge.org/core/books/quantum-computation-and-quantum-information/01E10196D0A682A6AEFFEA52D53BE9AE) | The bible, textbook | Paid |
| [NIST PQC Standards](https://csrc.nist.gov/projects/post-quantum-cryptography) | Standards docs | Free |
| [CISA Post-Quantum Cryptography Guide](https://www.cisa.gov/quantum) | Government guidance | Free |
| [Scott Aaronson's Blog — Shtetl-Optimized](https://scottaaronson.blog/) | Expert commentary | Free |

---

!!! tip "First Principles Summary"
    Quantum ≠ magic. It is **linear algebra applied to complex probability amplitudes**, with physical constraints (no cloning, no measurement without collapse). The security implications are real but the timeline is uncertain. Learn PQC now — it will be relevant before most RSA keys expire.
