# Spezzatura Engine — T² Specification

The Spezzatura Engine is the deterministic reputational core of F2F-RaaT.

It models reputation across six hybrid vectors:

- `C` — Contextual Compliance
- `A` — Operational Activity
- `T` — Time and regularity
- `U` — Uniqueness and exclusivity
- `R` — Relationships and clustering
- `Â` — Animus / operational intent

Core formula:

```text
score = log₂(C × A × T² × U × R × Â)
```

Outputs are typically mapped into discrete verdict bands (e.g., BLOCK / REVIEW / ALLOW).

**For the formal origin and calculation method of the input vectors (C, A, T, U, R, Â), see the [Vector Calculation Specification](vector_calculation.md).**
