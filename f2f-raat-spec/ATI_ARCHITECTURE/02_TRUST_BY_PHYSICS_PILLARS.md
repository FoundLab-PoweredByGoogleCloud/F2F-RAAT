# 2. The Three Pillars of Trust by Physics: The Veritas 2.0 Architecture

FoundLab's canonical architecture resolves the Retention Paradox by moving from a strategy of mitigating risk to one of **eliminating risk architecturally**. This is achieved by physically and cryptographically decoupling the *proof of an event* from the *data of an event*.

This is **Trust by Physics**, built on three core pillars.

---

### Pillar 1: Zero-Persistence — Eliminating Risk at the Source

**The safest data is data that no longer exists.** The Zero-Persistence doctrine dictates that sensitive data is only ever processed in volatile memory (RAM), never stored at rest.

- **Mechanism:** Sensitive data is processed within ephemeral, kernel-level isolated sandboxes (Google Cloud Run with gVisor). Upon completion of the task (e.g., an AI inference), the container is instantly destroyed (`SIGKILL`), and the memory is zeroed out.
- **Mandate Solved:** This satisfies the LGPD/GDPR "Right to be Forgotten" **by design**. There is no data to delete because it was never permanently stored. This is "Compliance-as-Infrastructure".

---

### Pillar 2: The Veritas Protocol — Mathematical Proof, Not Manual Process

In place of fallible human processes, Veritas provides an immutable, mathematical record that an event occurred and a decision was made, without holding the toxic data itself.

- **Mechanism:** Every decision generates a W3C Verifiable Credential, whose cryptographic hash is recorded in an immutable **WORM Ledger** (Write-Once-Read-Many), built on Google BigQuery with Bucket Lock. IAM roles prevent any modification or deletion of records.
- **Mandate Solved:** This satisfies the BACEN/SOX retention requirement by preserving a verifiable, tamper-proof audit trail for the required 5-10 year period.

---

### Pillar 3: Crypto-Shredding — The Right to Erasure, The Mandate to Retain

This is the engineering keystone that solves the paradox's final conflict. It allows a record to be both retained and "deleted" simultaneously.

- **Mechanism:** The data written to the WORM ledger is encrypted using a Customer-Managed Encryption Key (CMEK) via Google Cloud KMS. To honor an erasure request, the client **destroys the CMEK**.
- **Mandate Solved:**
    - The encrypted data (**ciphertext**) remains physically in the WORM ledger, satisfying the **retention mandate** (BACEN/SOX).
    - Without the key, the ciphertext becomes **irrecoverable mathematical entropy**—"cryptographic garbage". This satisfies the **erasure mandate** (LGPD).

> **Metaphor:** The data is in a locked drawer (WORM ledger), satisfying the auditor. The key to that drawer has been vaporized (Crypto-Shredding), satisfying the privacy regulator.

**Next:** [Cognitive Auditability & AI](./03_COGNITIVE_AUDITABILITY_AI.md)
