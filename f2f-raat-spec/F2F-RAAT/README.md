# F2F-RAaT: The Execution Contract
> **The normative rules of engagement for the decision engine.**

This directory defines the **laws** that the F2F-RAaT engine must obey. Unlike typical software where rules are implicit in code, here they are explicit in the **Execution Contract**.

## 📜 Normative Documents

| Document | Purpose |
| :--- | :--- |
| **[Execution Contract v1.1](F2F-RAAT_EXECUTION_CONTRACT_v1.1.md)** | **The Supreme Law.** Defines scope of authority, forbidden actions (MUST NOT), and authorized behaviors. |
| **[Invariants](invariants.md)** | Mathematical properties that must logically hold true (e.g., "Output Boundedness"). |
| **[Decision Model](decision_model.md)** | How inputs (Fact, Capsule) are mapped to outputs. |
| **[Economic Binding](economic_binding.md)** | Mechanisms for enforcing real-world consequences (staking, slashing, friction). |
| **[Policy Governance](policy_governance.md)** | Who is allowed to change the rules and how changes are propagated. |
| **[Compliance Mapping](../COMPLIANCE_KIT/COMPLIANCE_MATRIX.md)** | How this contract satisfies "Explainability" requirements. |

---

## 🔗 Architecture Context

F2F-RAaT operates within the broader **Auditable Trust Infrastructure (ATI)**.
See the [Main Architecture](../../ATI_ARCHITECTURE/) for the orchestration layer.

[⬅️ Back to Main Spec](../README.md)
