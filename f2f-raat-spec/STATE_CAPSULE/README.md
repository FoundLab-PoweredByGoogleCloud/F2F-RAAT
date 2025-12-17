# The State Capsule
> **"Reputation without Identity. Verification without Exposure."**

The **State Capsule** is the fundamental data structure of the ATI. It acts as a **PII-Free Boundary Object** that allows reputation to be ported, verified, and executed upon without ever touching the user's private data.

---

## 🔒 The Core Guarantee: "Zero-Knowledge Reputation"

The Capsule solves the privacy-utility paradox in compliance:
*How do I prove I trust this user to another bank without sending them the user's transaction history?*

**Answer:** You send the **State Capsule**.

| Contains (✅) | Does NOT Contain (❌) |
| :--- | :--- |
| **Reputational Scores** (T², P(x)) | **Names, Emails, Phones** (PII) |
| **Risk Vectors** (Velocity, Geo-Entropy) | **Raw Transaction Logs** |
| **Policy Flags** (KYC_TIER_3) | **Document Images / IDs** |
| **Cryptographic Proofs** (Merkle Roots) | **Credit Card Numbers** |

---

## 📦 Capsule Schema Structure

The capsule is a strictly typed JSON object.

```json
{
  "T²": {
    "score": 7.4,
    "vectors": { "C": 0.98, "T": 0.85, ... }
  },
  "P(x)": {
    "delta": 0.0,
    "decay_rate": 0.1
  },
  "Flags": ["KYC_VERIFIED", "DEVICE_TRUSTED"],
  "Tokens": ["XAI_REASON_CODE_404"],
  "Proof": "0x8f7d...",
  "Context": {
    "namespace": "br.fintech.pix",
    "timestamp": "2025-12-16T10:00:00Z"
  }
}
```

---

## 🛠️ Normative Specifications

- **[Capsule Schema (JSON)](capsule_schema.json)**: The strict validation schema.
- **[Constraint Logic](constraints.md)**: Rules for capsule validity (TTL, Origin).
- **[Validation Examples](validation_examples/)**: Test vectors for implementers.
- **[Compliance Mapping](../COMPLIANCE_KIT/COMPLIANCE_MATRIX.md)**: How the Capsule satisfies Privacy by Design/Default (GDPR Art. 25).

---

[⬅️ Back to Main Spec](../README.md)
