# Threat Model & Mitigations
> **"Assume the adversary is inside the system."**

This directory details the adversarial vectors against F2F-RAaT and the architectural mitigations ("Trust by Physics") that neutralize them.

We do not rely on "security through obscurity". We rely on mathematical constraints.

---

## 🛡️ Top-Level Threat Matrix

| Threat | Attack Vector | Mitigation Strategy |
| :--- | :--- | :--- |
| **Model Poisoning** | Feeding fake "good" facts to boost score. | **T² Temporal Weighting**. Trust ($T^2$) grows slowly; attacks must be sustained for years to work. |
| **Capsule Forgery** | Editing the JSON to change the score. | **Cryptographic Signing**. Capsules are signed by the issuer; any bit flip invalidates the sig. |
| **Inference Inference** | Reverse-engineering PII from the score. | **Bounded Output**. Decision bands (Block/Allow) leak almost zero bits of information about the input. |
| **Policy Capture** | Rogue admin changing rules to Allow attackers. | **Multi-Sig Governance**. Policy changes require consensus signatures. |
| **Oracle Corruption** | Lying about the input fact. | **Fact Formalization**. Facts are typed and signed at the source. |

---

## 🛑 The "Break-Glass" Protocol

What if the system blocks a VIP or fails critically?

F2F-RAaT implements a **Cryptographic Break-Glass**:
1.  Human overrides are permitted ONLY by signing a specific `OverrideToken`.
2.  This token is appended to the user's permanent `VeritasProof` chain.
3.  **Consequence**: You can override the AI, but you cannot hide that you did it.

## 📚 Detailed Analyses

- **[Adversarial Vectors](adversarial_vectors.md)**: Deep dive into attack scenarios.
- **[Mitigation Patterns](mitigation_patterns.md)**: Architectural defenses.

---

[⬅️ Back to Main Spec](../README.md)
