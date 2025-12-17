# F2F-RAaT Compliance Matrix
> **"Compliance-as-Infrastructure"**

This document maps the architectural pillars of F2F-RAaT to specific regulatory articles in **GDPR (EU)**, **LGPD (Brazil)**, and **BACEN (Central Bank of Brazil)**.

---

## 1. The Right to Erasure (Right to be Forgotten)

| Regulation | Article | Requirement | F2F-RAaT Solution |
| :--- | :--- | :--- | :--- |
| **GDPR** | Art. 17 | Data subjects have the right to obtain erasure of personal data without undue delay. | **Zero-Persistence Doctrine**: Data is never stored at rest in the first place. It is processed in ephemeral RAM and destroyed immediately. |
| **LGPD** | Art. 18 (VI) | Holder may petition for elimination of unnecessary or excessive data. | **Crypto-Shredding**: Even if logs exist, destroying the CMEK key renders them mathematically non-existent (garbage entropy). |

---

## 2. Auditability & Explanability

| Regulation | Article | Requirement | F2F-RAaT Solution |
| :--- | :--- | :--- | :--- |
| **BACEN** | Res. 4.893 | Audit trails must be kept for 5-10 years for inspection. | **Veritas Protocol**: Immutable, hash-linked WORM ledger guarantees history cannot be rewritten. |
| **GDPR** | Art. 22 | Right to explanation of automated decision making. | **Spezzatura Vectors**: Every score is a deterministic product of explicit facts. No "black box" AI. We can trace the exact line of code (policy) that caused a decision. |

---

## 3. Data Minimization & Privacy by Design

| Regulation | Article | Requirement | F2F-RAaT Solution |
| :--- | :--- | :--- | :--- |
| **GDPR** | Art. 25 | Implement technical measures to ensure only necessary data is processed. | **State Capsule**: The engine sees only the `Capsule` (scores/flags), never the raw PII or identity documents. |
| **LGPD** | Art. 46 | Adopt security, technical and administrative measures to protect personal data. | **Kernel-Level Isolation**: Usage of gVisor sandboxes ensures workload isolation at the syscall layer. |

---

## 4. Governance & Integrity

| Regulation | Article | Requirement | F2F-RAaT Solution |
| :--- | :--- | :--- | :--- |
| **BACEN** | Res. 4.968 | Internal controls and governance structures. | **Multi-Sig Policy**: Changes to the rules require M-of-N consensus, preventing unilateral insider modification. |
| **SOX** | Sec. 404 | Internal control report accreditation. | **Veritas Proofs**: Every decision emits a cryptographic proof that links Policy Version + Input + Output. |
