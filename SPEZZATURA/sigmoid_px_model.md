# Sigmoid P(x) Reactivity Model

This document specifies the operational model for the Sigmoid P(x) score, which serves as the framework's acute reactivity engine.

---

### 1. Triggers: High-Severity Facts

The P(x) score is triggered exclusively by **Facts** with a `severity` value greater than or equal to a threshold defined in the `POLICY`.

Examples of fact types that typically trigger the P(x) model:
- `geo_conflict`
- `velocity_anomaly`
- `structural_break`
- `failed_auth_chain`
- `contextual_incoherence`

---

### 2. Operational Formula

The change in P(x) is calculated based on the event's severity and the time elapsed since the event.

```
ΔP(x) = sigmoid(α * severity - β * time_since_event)
```
Where `α` and `β` are coefficients defined in the `POLICY` to control the sensitivity and time-decay influence.

---

### 3. Decay Model

The effect of a P(x) adjustment is temporary and decays exponentially to avoid permanent contamination of the reputational score.

- **Effect Window**: The effect lasts for a fixed duration (e.g., 15 minutes, 1 hour) as defined by `Policy.reactivity_window`.
- **Decay Formula**: Within the window, the score decays over time `Δt`:
  ```
  P(x)_new = P(x)_old * exp(-λ * Δt)
  ```
  Where `λ` is the decay constant, also defined in the `POLICY`.

The core principle is that P(x) temporarily adjusts the overall risk profile without altering the structural T² score.
