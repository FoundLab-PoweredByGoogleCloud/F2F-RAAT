# Scenario 1: The High-Frequency Pix Attack
> **Context**: A compromised account attempts to drain funds via small, rapid Pix transactions to avoid traditional bulk-value limits.

---

## 1. The Setup (T = 0s)
**User State**: Valid account, good history.
**Attacker Goal**: Move R$ 50.000 in 100 transactions of R$ 500.

**Initial Capsule State**:
```json
{
  "T²": { "score": 8.0, "vectors": { "A": 0.5, "Velocity": 0.1 } },
  "Flags": []
}
```

---

## 2. The Attack Begins (T = 0s to T = 2s)
The attacker scripts 5 transactions of R$ 500 in 2 seconds.

*   **Fact 1**: `Transfer R$ 500` -> **Decision**: `ALLOW`. (Normal behavior).
*   **Fact 2**: `Transfer R$ 500` -> **Decision**: `ALLOW`.
*   **Fact 3**: `Transfer R$ 500` -> **Decision**: `ALLOW`.

**Engine Reaction**:
With each execution, the `Activity (A)` vector and the hidden `Velocity` counter in the capsule increase slightly. The T² score dips slightly due to "Abnormal Frequency", but remains in the Green Zone.

---

## 3. The Tipping Point (T = 3s)
Transaction #6 arrives. Ideally, a human analyst would notice this pattern 2 hours later.
**F2F-RAaT notices it in 3ms.**

*   **Fact 6**: `Transfer R$ 500`.
*   **Capsule Check**: Velocity Vector crosses the `SIGMOID_LIMIT_ACUTE` threshold defined in Policy.
*   **Spezzatura Calc**: The score crashes from 7.8 to 4.1 instantly due to the `U (Uniqueness)` and `A (Activity)` vectors colliding.

---

## 4. The Binding Execution
The Burn Engine sees the score drop below 4.5 (`BLOCK_THRESHOLD`).

**Output Decision**:
```json
{
  "decision": "BLOCK_AND_FREEZE",
  "reason": "VELOCITY_ANOMALY_HFT",
  "veritas_proof": "0x99aad...",
  "effects": {
    "freeze_duration": "24h",
    "challenge_required": "BIOMETRIC_LIVENESS"
  }
}
```

**Result**: The attack is stopped at R$ 3.000 loss, instead of R$ 50.000.
The user's Capsule is now marked with a `FREEZE` flag. No further transactions are possible until the specific challenge (Biometrics) is met.
