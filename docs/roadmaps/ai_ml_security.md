# AI/ML Security Roadmap

**Goal:** Understand how machine learning systems are attacked and defended — from adversarial inputs to model supply chains to LLM exploitation.

!!! abstract "fps advice"
    AI/ML security is not a separate field. It is the intersection of software security, mathematics, and systems thinking. If you understand how models work mechanically, you understand how they break.

---

## Why This Matters Now

Every major application is integrating ML. Authentication systems use facial recognition. Email filters use NLP classifiers. Code assistants write production code. Autonomous systems make physical decisions. Each integration creates attack surface that most security teams are not equipped to evaluate.

The attackers who understand ML internals will find vulnerabilities invisible to traditional security practitioners. The defenders who understand ML internals will catch threats that scanners miss entirely.

---

## Phase 0 — Foundations

You need these before ML security makes sense. No shortcuts.

### Mathematics

- **Linear algebra** — vectors, matrices, dot products, eigenvalues. This is the language of ML.
- **Calculus** — gradients, partial derivatives, chain rule. This is how models learn.
- **Probability & statistics** — distributions, Bayes' theorem, hypothesis testing. This is how models reason.

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [3Blue1Brown — Essence of Linear Algebra](https://www.youtube.com/playlist?list=PLZHQObOWTQDPD3MizzM2xVFitgF8hE_ab) | Video | Best visual intuition for linear algebra |
| [3Blue1Brown — Neural Networks](https://www.youtube.com/playlist?list=PLZHQObOWTQDNU6R1_67000Dx_ZCJB-3pi) | Video | How neural nets actually work, visually |
| [Khan Academy — Linear Algebra](https://www.khanacademy.org/math/linear-algebra) | Interactive | Practice problems + theory |
| [StatQuest](https://www.youtube.com/c/joshstarmer) | Video | Statistics explained without jargon |

### Python + ML Basics

- Comfortable with Python (NumPy, pandas)
- Understand what a neural network does mechanically (forward pass, backprop, loss function)
- Can train a simple model using PyTorch or TensorFlow

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [fast.ai — Practical Deep Learning](https://course.fast.ai/) | Course | Best hands-on ML course, free, top-down approach |
| [Andrej Karpathy — Neural Networks: Zero to Hero](https://www.youtube.com/playlist?list=PLAqhIrjkxbuWI23v9cThsA9GvCAUhRvKZ) | Video | Build a neural net from scratch in Python |
| [PyTorch Tutorials](https://pytorch.org/tutorials/) | Docs | Official, well-structured |

### Security Fundamentals

- Web security basics (OWASP Top 10)
- Basic threat modeling
- Understanding of supply chain attacks

→ If you need this: [Cybersecurity Roadmap](cybersecurity.md)

---

## Phase 1 — Adversarial Machine Learning

This is where security meets ML. The core question: **how do models fail when someone is actively trying to make them fail?**

### Adversarial Examples

Inputs crafted to fool models while appearing normal to humans. A pixel-level perturbation can flip a classifier's output with high confidence.

**Key concepts:** FGSM (Fast Gradient Sign Method), PGD (Projected Gradient Descent), C&W attack, transferability of adversarial examples across models.

**Why it matters:** If your authentication uses facial recognition, an adversarial patch on glasses can bypass it. If your content moderation uses a classifier, adversarial text can evade it.

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [Adversarial Robustness Toolbox (ART)](https://github.com/Trusted-AI/adversarial-robustness-toolbox) | Tool | IBM's library — generate and defend against adversarial attacks |
| [CleverHans](https://github.com/cleverhans-lab/cleverhans) | Tool | Reference implementations of adversarial attacks |
| [Goodfellow et al. — Explaining and Harnessing Adversarial Examples](https://arxiv.org/abs/1412.6572) | Paper | The foundational FGSM paper |
| [Madry et al. — Towards Deep Learning Models Resistant to Adversarial Attacks](https://arxiv.org/abs/1706.06083) | Paper | PGD and adversarial training |

### Data Poisoning

Corrupting training data to control model behavior. If you can influence what a model learns from, you control what it does.

**Key concepts:** backdoor attacks (trigger patterns), label flipping, clean-label attacks.

### Model Extraction & Stealing

Querying a black-box model systematically to reconstruct its behavior. If an API exposes predictions, an attacker can approximate the model without access to weights.

---

## Phase 2 — LLM Security

Large language models have their own attack surface. This is the fastest-moving area in all of security right now.

### Prompt Injection

The SQL injection of the AI era. Untrusted input that manipulates model behavior.

- **Direct injection** — user crafts input that overrides system instructions
- **Indirect injection** — malicious content embedded in data the LLM processes (web pages, emails, documents)

### Key Threat Areas

- **Jailbreaking** — bypassing safety filters and alignment
- **Data exfiltration** — extracting training data or system prompts through crafted queries
- **Agent exploitation** — manipulating LLM-powered agents that can take actions (tool use, code execution)
- **Supply chain** — poisoned fine-tuning data, backdoored model weights on public hubs

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [OWASP Top 10 for LLM Applications](https://genai.owasp.org/) | Framework | Standardized LLM vulnerability classification |
| [MITRE ATLAS](https://atlas.mitre.org/) | Framework | ATT&CK-style matrix for AI/ML threats |
| [Simon Willison's blog](https://simonwillison.net/series/prompt-injection/) | Blog | Best ongoing coverage of prompt injection |
| [Lakera Gandalf](https://gandalf.lakera.ai/) | Interactive | Practice prompt injection challenges |
| [Garak](https://github.com/NVIDIA/garak) | Tool | LLM vulnerability scanner |
| [rebuff](https://github.com/protectai/rebuff) | Tool | Prompt injection detection framework |

---

## Phase 3 — ML Supply Chain & Infrastructure

### Model Supply Chain

Public model hubs (Hugging Face, PyTorch Hub) are the new npm. Models can contain:

- Arbitrary code execution via pickle deserialization
- Backdoored weights that activate on specific triggers
- Poisoned tokenizers or preprocessing pipelines

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [Hugging Face Security](https://huggingface.co/docs/hub/en/security) | Docs | Model hub security practices |
| [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence) | Framework | Federal AI risk guidance |
| [ProtectAI — Model Security](https://protectai.com/) | Platform | ML supply chain security tools |

### MLOps Security

Training pipelines, data stores, model registries, inference endpoints — each is attack surface. Traditional infra security applies but with ML-specific nuances: GPU cluster access, experiment tracking, feature stores, model versioning.

---

## Phase 4 — Hands-On Practice

| Platform | Focus | Cost |
| :--- | :--- | :--- |
| [Lakera Gandalf](https://gandalf.lakera.ai/) | Prompt injection challenges | Free |
| [Damn Vulnerable LLM Agent](https://github.com/WithSecureLabs/damn-vulnerable-llm-agent) | LLM agent exploitation | Free |
| [HackAPrompt](https://huggingface.co/spaces/jerpint-org/HackAPrompt) | Prompt injection CTF | Free |
| [Adversarial Robustness Toolbox](https://github.com/Trusted-AI/adversarial-robustness-toolbox) | Adversarial ML experiments | Free |
| [MLSecOps Top 10](https://owasp.org/www-project-machine-learning-security-top-10/) | Vulnerability checklist | Free |

---

## References

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [MITRE ATLAS](https://atlas.mitre.org/) | Framework | The ATT&CK for AI |
| [OWASP LLM Top 10](https://genai.owasp.org/) | Framework | LLM vulnerability standard |
| [NIST AI RMF](https://www.nist.gov/artificial-intelligence) | Framework | Federal AI risk guidance |
| [Anthropic Research](https://www.anthropic.com/research) | Research | Alignment and safety research |
| [OpenAI Security](https://openai.com/security) | Research | Red teaming and model safety |

---

!!! abstract "fps advice"
    The field is moving faster than any documentation can keep up. Anchor your understanding in the math and security fundamentals. The specific tools and attacks will change every quarter. The principles — adversarial thinking, threat modeling, supply chain verification — are permanent.