# Spezzatura Vector Calculation

This document specifies the formal origin and calculation method for the six input vectors of the Spezzatura T² engine.

The vectors are not derived from raw data but from typed **Facts** aggregated over a policy-defined time window. Each vector is a deterministic function:

`Vector = f(Facts[domain], TimeWindow, Constraints, Policy)`

---

### C — Conformity

- **Scale**: `[0.0, 1.0]` (higher is better)
- **Source Facts**: `policy_violation`, `compliance_event`, `rule_conflict`, `risk_mismatch`.
- **Calculation Logic**: The Conformity score starts at 1.0 and is penalized by the rate of violations within a given time window.
  ```
  C = 1.0 - (penalty_rate * number_of_violations_in_window)
  ```
  The `penalty_rate` is defined in the `POLICY`.

---

### A — Activity

- **Scale**: `[0.0, 1.0]`
- **Source Facts**: `transaction_event`, `access_event`, `engagement_event`.
- **Calculation Logic**: The Activity score is a normalized measure of operational frequency against an expected baseline.
  ```
  A = normalize(activity_count / baseline_expected_in_window)
  ```
  The `baseline_expected` is defined in the `POLICY`.

---

### T — Time (Regularity)

- **Scale**: `[0.0, 1.0]`
- **Source Facts**: `fact.sequence`, `monotonic_timestamp`.
- **Calculation Logic**: The Time vector measures the regularity and predictability of events, not just their age. A high score indicates a consistent, non-erratic pattern.
  ```
  T = regularity_score
  ```
  The `T²` component is applied only within the final Spezzatura formula.

---

### U — Uniqueness

- **Scale**: `[0.0, ∞)` (higher is better)
- **Source Facts**: Analysis of patterns across multiple facts.
- **Calculation Logic**: The Uniqueness score measures the entropy and diversity of behavioral patterns, rewarding non-repetitive or non-sybil-like activity.
  ```
  U = uniqueness_entropy_score
  ```

---

### R — Relations

- **Scale**: `[0.0, ∞)` (higher is better)
- **Source Facts**: Relational or graph-based facts linking entities.
- **Calculation Logic**: The Relations score is derived from the subject's centrality in a reputational graph, indicating its connections to other high-reputation entities.
  ```
  R = centrality_score_of_subject
  ```

---

### Â — Ânimo (Operational Intent)

- **Scale**: `[0.0, 1.0]` (higher is better)
- **Source Facts**: `incoherence_fact`, `failed_attempt_sequence`, `suspicious_sequence`.
- **Calculation Logic**: The Ânimo vector measures the consistency of actions over time. A high score reflects a clear, coherent operational pattern, while a low score indicates erratic, contradictory, or suspicious sequences.
  ```
  Â = f(consistency_over_time)
  ```
