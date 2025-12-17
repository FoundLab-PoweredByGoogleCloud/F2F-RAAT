# F2F-RAaT Repository Master Index
> **"Guide to the Spec."**

Use this index to navigate the F2F-RAaT specification based on your role.

---

## 👔 For Executives & Stratagy
*Start here to understand the "Why" and the "Value".*

*   **[The Whitepaper (Executive Summary)](WHITEPAPER/README.md)**: The high-level theory.
*   **[Strategic Impact](ATI_ARCHITECTURE/04_STRATEGIC_IMPACT.md)**: How this tech creates competitive advantage ("Operational Alpha").
*   **[The Regulatory Paradox](ATI_ARCHITECTURE/01_THE_REGULATORY_PARADOX.md)**: The compliance problem we solve.

## 🏗️ For Architects & Product Owners
*How the system is built and behaves.*

*   **[Architecture Overview (The 3 Pillars)](ATI_ARCHITECTURE/02_TRUST_BY_PHYSICS_PILLARS.md)**: Zero-Persistence, WORM, Crypto-Shredding.
*   **[Cookbook: Pix Fraud Attack](COOKBOOKS/01_pix_hft_attack.md)**: Real-world scenario walkthrough.
*   **[Cookbook: Merchant Trust Growth](COOKBOOKS/02_merchant_gradual_trust.md)**: How reputation scales over time.
*   **[Governance & Policy](GOVERNANCE/README.md)**: Who controls the rules.

## 💻 For Engineers & Developers
*How to implement and integrate.*

*   **[API Specification (OpenAPI)](INTERFACE_CONTRACTS/f2f-raat-api.yaml)**: The REST contract.
*   **[Input/Output Vectors](COMPLIANCE_KIT/compliance_vectors_v1.json)**: Test cases for QA.
*   **[State Capsule Schema](STATE_CAPSULE/capsule_schema.json)**: The JSON structure of reputation.
*   **[Fact Schemas](INTERFACE_CONTRACTS/FACT_SCHEMAS/)**: JSON Schemas for standard input facts.
*   **[Burn Engine Logic](BURN_ENGINE/burn_engine_spec.md)**: How decisions are computed.

## 🛡️ For Auditors & Security
*How to prove it is safe and compliant.*

*   **[Veritas Proof Spec](VERITAS/veritas_proof_spec.md)**: How the audit trail works.
*   **[Threat Model](THREAT_MODEL/README.md)**: Defense against poisoning and attacks.
*   **[Compliance Matrix](COMPLIANCE_KIT/COMPLIANCE_MATRIX.md)**: Regulatory mapping (GDPR/LGPD/BACEN).
*   **[Cognitive Auditability](ATI_ARCHITECTURE/03_COGNITIVE_AUDITABILITY_AI.md)**: Proving AI intent.

---

## 📂 Directory Structure

```text
/
├── ATI_ARCHITECTURE/    # The Ecosystem (Strategy)
├── BURN_ENGINE/         # The Decision Logic
├── COMPLIANCE_KIT/      # Test Vectors (QA)
├── COOKBOOKS/           # Real-world Scenarios
├── F2F-RAAT/            # The Execution Contract
├── GOVERNANCE/          # Admin & Control
├── INTERFACE_CONTRACTS/ # API Specs (OpenAPI)
├── SPEZZATURA/          # The Math (T² Model)
├── STATE_CAPSULE/       # The Data Structure
├── THREAT_MODEL/        # Security Analysis
├── VERITAS/             # The Audit Trail
└── WHITEPAPER/          # Theory & Glossary
```
