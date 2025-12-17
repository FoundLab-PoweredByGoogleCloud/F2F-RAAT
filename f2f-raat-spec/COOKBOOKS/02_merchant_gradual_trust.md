# Scenario 2: The New Merchant (Trust takes Time)
> **Context**: A new e-commerce merchant ("Loja Legal") joins the platform. They want to process R$ 100k/day, but they have zero history.

---

## 1. Day 1: The Zero State
New entities start with a **Golden Seed** capsule, but with low limits.

*   **Initial T² Score**: 5.0 (Review Zone).
*   **Vector T (Time)**: 0.1 (Very low).
*   **Policy Limit**: R$ 5k/day.

**Merchant Action**: Processes R$ 2k in sales.
**Engine Reaction**: `ALLOW`. The transactions are small and diverse (`Uniqueness` is high).
**Capsule Update**: `T` increases slightly. `Activity` is healthy.

---

## 2. Day 30: Consistency is Key
The merchant has transacted every day for 30 days without chargebacks.

*   **Vector T (Time)**: The $T^2$ component in the formula kicks in.
    *   $Latex: Score = \log(C \times A \times T^2 ...)$
    *   Since $T$ is squared, consistent behavior boosts the score faster than volume.
*   **New Score**: 6.2 (Allow Zone).
*   **New Policy Limit**: Auto-adjusted to R$ 50k/day.

---

## 3. Day 90: The Attempted "Exit Scam"
The merchant tries to process R$ 500k in one hour (10x their normal volume).

*   **Fact**: `Volume Spike R$ 500k`.
*   **Spezzatura Calc**:
    *   Vector `A` (Activity) spikes massively.
    *   Vector `C` (Compliance) drops because this violates the "Consistent Growth" pattern.
    *   The engine detects a divergence between `T` (Long term trust) and `A` (Short term burst).

**Output Decision**: `ESCROW_OR_HOLD`.
The funds are processed but **held** for 7 days to wait for chargebacks. The system trusts the merchant enough not to block, but not enough to release half a million instantly.

**Result**: The merchant cannot run away with the money. Trust is enforced by physics.
