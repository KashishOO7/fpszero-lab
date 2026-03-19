# AI/ML Security

> Machine learning models are software. Software has bugs. The bugs in ML systems are just harder to see.

AI/ML security sits at the intersection of machine learning, adversarial mathematics, and traditional security. As ML systems are deployed in authentication, content moderation, autonomous vehicles, and financial decisions, the attack surface is expanding faster than the defenses.

---

## Why This Matters

!!! abstract "fps advice"
    AI/ML security is not a future problem. Models are in production now — making decisions about credit, identity, content, and access. Understanding how they fail is understanding how the systems around you actually work.

---

## The Threat Landscape

| Attack Category | What It Is | Example |
| :--- | :--- | :--- |
| **Adversarial examples** | Inputs crafted to fool a model while looking normal to humans | A sticker on a stop sign that makes a vision model read "speed limit 45" |
| **Data poisoning** | Corrupting training data to influence model behavior | Injecting mislabeled samples into a public dataset used for training |
| **Model extraction** | Stealing a model's behavior through API queries | Querying a black-box API systematically to reconstruct the model |
| **Membership inference** | Determining if a specific data point was in the training set | Checking if your medical record was used to train a health model |
| **Prompt injection** | Manipulating LLM behavior through crafted input | Embedding instructions in web content that an LLM-powered agent reads |
| **Model inversion** | Reconstructing training data from model outputs | Recovering faces from a facial recognition model's gradients |

---

## Core Concepts

**Adversarial robustness** is the measure of how much a model's predictions change under small, deliberate input perturbations. Most neural networks are surprisingly fragile — a few pixels changed in an image can flip a classification with high confidence.

**Supply chain attacks on ML** target the training pipeline: poisoned datasets, compromised pre-trained models, backdoored model weights distributed through public repositories.

**Privacy in ML** involves differential privacy, federated learning, and the tension between model utility and data protection. Models memorize training data more than most people realize.

---

## Resources

| Resource | Type | Why It's Here |
| :--- | :--- | :--- |
| [MITRE ATLAS](https://atlas.mitre.org/) | Framework | ATT&CK-style framework specifically for ML threats |
| [Adversarial Robustness Toolbox (ART)](https://github.com/Trusted-AI/adversarial-robustness-toolbox) | Tool | IBM's library for testing ML model robustness |
| [OWASP ML Security Top 10](https://owasp.org/www-project-machine-learning-security-top-10/) | Reference | Standardized ML vulnerability classification |
| [NIST AI Risk Management Framework](https://www.nist.gov/artificial-intelligence) | Framework | Federal guidance on AI risk |
| [Hugging Face — ML Security](https://huggingface.co/docs/hub/en/security) | Reference | Security considerations for model distribution |

---

## Landscape

- Adversarial attacks on computer vision systems
- LLM jailbreaking and prompt injection defense
- Deepfake detection and generation
- Watermarking and provenance for AI-generated content
- Secure ML inference (homomorphic encryption, secure enclaves)
- AI-powered offensive security tooling

---

!!! abstract "fps advice"
    AI/ML security is one of the fastest-moving domains in all of security. Resources go stale quickly. Anchor your understanding in the math (linear algebra, optimization, probability) — the fundamentals don't change even when the tools do.
