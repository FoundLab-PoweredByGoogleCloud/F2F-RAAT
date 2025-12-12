# Burn Engine Specification

The Burn Engine is the deterministic authorization runtime consuming:

- `FACT`
- `STATE_CAPSULE`
- `POLICY`
- `RUNTIME_VERSION`

It MUST:

- Use deterministic, bounded evaluation.
- Select a decision from a finite, policy-defined set.
- Emit Veritas commitments for every decision.

---

## Effect Parameterization

While the Burn Engine selects the *type* of decision (e.g., `RATE_LIMIT`, `ESCROW_OR_HOLD`), the specific **parameters** for that decision are defined exclusively and immutably within the active `POLICY`. The runtime does not calculate these parameters.

### Formal Policy Example

The `POLICY` object contains a dedicated structure for defining these parameters:
```json
"POLICY": {
  "effects": {
    "RATE_LIMIT": {
        "limit_window": "1h",
        "max_ops": 10
    },
    "ESCROW_OR_HOLD": {
        "duration": "24h",
        "release_conditions": ["no_new_flags"]
    },
    "ALLOW_WITH_CONDITIONS": {
        "step_up": "proof_of_ownership",
        "validity": "6h"
    }
  }
}
```

### Execution Logic

The Burn Engine's execution logic is therefore:
```
decision_type = f(FACT, STATE_CAPSULE, POLICY)
parameters = POLICY.effects[decision_type]
apply_decision(decision_type, parameters)
```
This ensures a complete separation of concerns between the decision logic and the policy configuration.
