# F2F-RAaT Whitepaper — v1.1
### From Fact to Feedback: A Reputation-as-a-Transaction Framework
### FoundLab — Auditable Trust Infrastructure

---

## 0. Executive Summary

**F2F-RAaT** is a deterministic, auditable, and mathematically verifiable reputational engine that transforms factual events into **executable, economically-binding decisions**—without statistical inference, predictive models, or data exposure.

It serves as the core trust runtime for **Auditable Trust Infrastructure (ATI)**.

At its heart is a simple, powerful pipeline:

```text
Fact → Reputational State → Executable Decision → Auditable Proof → Feedback → New State
```

This is reputation as a formal mechanism: no databases, no ETL, no heuristics, and no guesswork. Reputation becomes execution. Execution becomes proof. Proof feeds back into reputation.

---

## 1. The Structural Problem with Reputation

Historically, reputation has been a passive, after-the-fact metric, suffering from critical flaws:

| Problem            | Consequence                     |
|--------------------|---------------------------------|
| **Passive Metric**   | Does not participate in decisions |
| **Post-Facto**       | Fails to influence the runtime    |
| **Human-Dependent**  | Subjective and non-auditable    |
| **Non-Deterministic**| Impossible to formally validate   |
| **Siloed**           | Not portable or interoperable   |
| **Lacks Proof**      | Creates legal & operational risk|

**The diagnosis is clear:** reputation has never been a mechanism for *execution*, only for delayed observation.

F2F-RAaT was designed to elevate reputation into a form of **auditable operational causality**.

---

## 2. The F2F-RAaT Cycle Architecture

The framework operates as a closed-loop system, ensuring that every action is based on prior state and contributes to future state. This cycle is the engine of its antifragility and auditability.

```mermaid
flowchart TD
  A[Formal Fact] --> B[Spezzatura T² Engine]
  B --> C[Reputational Effects (Sigmoid, Tokens)]
  C --> D[State Capsule Assembly]
  D --> E[Burn Engine (Decision)]
  E --> F[Binding Decision]
  F --> G[Sealed Rationale + Veritas Proof]
  G --> H[Reputational Feedback]
  H --> B
```

The architectural layers are as follows:

1. **Fact Layer** — fato formal, tipado, sem PII.  
2. **Effect Layer** — Spezzatura T² + Sigmoid P(x) + Tokens/Flags.  
3. **State Capsule** — boundary reputacional PII-free, não-Turing, com proofs.  
4. **Decision Layer (Burn Engine)** — execução determinística e bounded.  
5. **Feedback Layer (Veritas)** — DecisionID, rationale selada, hash-chain.  
6. **Closed Loop** — atualização do estado reputacional (antifragilidade).

---

## 3. The Fact Layer: Formal Facts

Example of a formal, PII-free fact:

```json
{
  "fact_id": "uuid",
  "type": "geo_conflict",
  "severity": 0.72,
  "timestamp": "2025-12-12T14:10Z"
}
```

Properties:
- No inference.
- No predictive models.
- No PII.
- No subjectivity—only objectified behavior.

---

## 4. The Effect Layer: Triple Reputation Engine

### 4.1 Spezzatura Engine (T²)

It models reputation across six hybrid vectors:

- **C** — Contextual Compliance
- **A** — Operational Activity
- **T** — Time and regularity
- **U** — Uniqueness and exclusivity
- **R** — Relationships and clustering
- **Â** — Animus / operational intent

Formula:
```text
score = log₂(C × A × T² × U × R × Â)
```

Typical verdict bands:
- `BLOCK`   → score ≤ 4.5
- `REVIEW`  → 4.5 < score ≤ 5.5
- `ALLOW`   → score > 5.5

Properties: deterministic, auditable, regulatory-friendly, and includes a sealed rationale hash.

### 4.2 Sigmoid P(x) Score

A reactivity engine for acute events that temporarily adjusts risk without contaminating the structural Spezzatura score.

### 4.3 Explainable Tokens + Flags

- Machine-readable XAI tokens.
- Reputational flags (e.g., policy violations, cluster behavior).
- These feed into the **State Capsule** construction.

---

## 5. The State Capsule: A Reputational Boundary

The capsule is a portable, verifiable artifact that carries reputational state, not raw data.

Generic example:
```json
{
  "T²": { "score": 5.8, "vectors": { "C": 0.9, "A": 0.8 } },
  "P(x)": { "delta": 0.12 },
  "Flags": ["GEO_MISMATCH_LOW"],
  "Tokens": ["GEO_CONFIDENCE_DELTA_MEDIUM"],
  "Proof": "0xdeadbeef...",
  "Constraints": { "jurisdiction": "BR" }
}
```

Properties:
- Contains zero PII.
- Is not Turing-complete (cannot execute code).
- Can be cryptographically verified.
- Portable across banks, blockchains, PSPs, and regulated domains.

It solves the **oracle problem**: reputation becomes verifiable without exposing the underlying source data.

---

## 6. The Decision Layer: The Burn Engine

The Burn Engine consumes `(FACT, STATE_CAPSULE, POLICY)` and emits a binding decision from a finite, policy-defined set:

- `ALLOW`
- `DENY`
- `ALLOW_WITH_CONDITIONS`
- `RATE_LIMIT`
- `ESCROW_OR_HOLD`
- `QUARANTINE`
- `FLAG_ONLY`
- `EMERGENCY_HALT` (if policy-enabled)

Guarantees:
- Deterministic-IO.
- Bounded-time and bounded-memory.
- Fail-closed (defaults to the safest outcome on failure).
- Zero-persistence (no intermediate state survives execution).

---

## 7. The Feedback Layer: Veritas & Sealed Rationales

Each decision generates:
- **DecisionID**
- **PolicyID**
- **FactDigest**
- **CapsuleHash**
- **RuntimeVersion**
- **Sealed Rationale (bounded-output XAI)**
- **VeritasProof (immutable hash-chain)**
- **ReversibilityToken** for authenticated overrides.

This allows regulators and auditors to:
- Re-execute the decision.
- Verify its integrity.
- Confirm that no PII was exposed.

---

## 8. Core Model Invariants

F2F-RAaT satisfies:
```text
Capsule ⊨ { No-Exfiltration ∧ Bounded-Output ∧ Deterministic-IO }
```
- **No-Exfiltration**: Capsules and rationales do not encode sensitive data.
- **Bounded-Output**: Decisions and explanations are from a finite set.
- **Deterministic-IO**: Same input → same output → same audit trail.

---

## 9. Threat Model (High-Level Overview)

Key vectors include:
- Inference leaks.
- Capsule poisoning.
- Side-channels in confidential computing.
- Policy capture / authority creep.
- Break-glass abuse.

Detailed mitigations are specified in the `THREAT_MODEL/` and `EXECUTION_CONTRACT/` components.

---

## 10. Conclusion

F2F-RAaT transforms reputation into a form of **computable physics**:

> A deterministic authorization engine with cryptographic proof, zero inference, zero persistence, and zero leakage, designed for high-risk, regulated environments.

This whitepaper defines the conceptual foundation. The formal, normative details are located in the **Execution Contract**.
