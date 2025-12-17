# F2F-RaaT Execution Contract — v1.0

This document defines the **formal execution boundaries**, **invariants** and  
**authorizations** for the F2F-RaaT runtime.

It SHALL be treated as normative specification for all compliant implementations.

---

## 1. Scope of Authority

### 1.1 Authorized Behavior (SHALL)

F2F-RaaT SHALL be authorized to:

- Compute and update reputational state transitions from submitted `FACT` and current `STATE_CAPSULE`.  
- Emit authorization decisions strictly as a function of `(FACT, STATE_CAPSULE, POLICY, RUNTIME_VERSION)` and deterministic runtime parameters.  
- Trigger bounded, pre-declared effects via the internal Burn Engine, each explicitly enumerated in `POLICY`.

### 1.2 Forbidden Behavior (MUST NOT)

F2F-RaaT MUST NOT be authorized to:

- Decide on legal conclusions (culpability, intent, fraud determination, regulatory classification, sanctions) or provide legal/financial advice.  
- Access, infer, reconstruct, or request PII, raw documents, free-text payloads, or external data sources.  
- Execute arbitrary code, initiate outbound network calls, or perform non-policy-defined actions.  
- Override ATI boundaries, bypass zero-persistence, or relax invariants under any circumstances.
