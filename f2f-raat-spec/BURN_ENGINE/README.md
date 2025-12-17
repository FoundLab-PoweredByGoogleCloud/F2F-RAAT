# The Burn Engine
> **"Finality is a feature. Decisions must close the loop."**

The **Burn Engine** is the deterministic actuator of F2F-RAaT. It consumes state and produces binding effects.

While other systems produce "alerts" for humans to review, the Burn Engine is designed to **execute**. It is the bridge between "knowing" (Risk State) and "doing" (Block/Allow).

---

## ⚙️ Execution Logic

The Engine operates as a pure function:

$$
f(\text{Fact}, \text{Capsule}, \text{Policy}) \rightarrow (\text{Decision}, \text{Effect}, \text{Proof})
$$

### The Decision Spectrum

The engine outputs one of the following bounded states:

| Decision | Meaning | Typical Use Case |
| :--- | :--- | :--- |
| **ALLOW** | Proceed normally. | Low risk transaction. |
| **ALLOW_WITH_CONDITIONS** | Proceed, but add friction. | High value transfer (requires 2FA). |
| **RATE_LIMIT** | Slow down execution. | Velocity violation. |
| **ESCROW_OR_HOLD** | Pause for time $t$. | Suspicious receiver. |
| **QUARANTINE** | Isolate user sandbox. | Device compromise. |
| **DENY** | Hard block. | Sanctions hit. |
| **EMERGENCY_HALT** | Circuit breaker. | System-wide attack. |

---

## 🚀 Key Properties

1.  **Deterministic IO**: Given the same Policy and Inputs, the decision is mathematically identical.
2.  **Fail-Closed**: If the engine cannot compute a safe state, it reverts to the safest policy (usually `DENY` or `HOLD`), never open.
3.  **Zero-Persistence**: The engine has no memory. It does not "learn". It only executes based on the Capsule provided.

## 📚 Specifications

- **[Burn Engine Spec](burn_engine_spec.md)**: The runtime logic constraints.
- **[Effect Set](effect_set.md)**: Detailed definitions of all possible actions.

---

[⬅️ Back to Main Spec](../README.md)
