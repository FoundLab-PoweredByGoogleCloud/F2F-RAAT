# OverrideFact Semantics

## The Non-Paradox of Deterministic Overrides

An `OverrideFact` is not a bypass of the F2F-RAaT pipeline. It is a special class of **Fact** with the highest normative priority, as defined by the active `POLICY`.

It does not break the deterministic model; it participates in it.

## Execution Flow

The standard execution is:
`Decision = f(Facts[], StateCapsule, Policy)`

When an `OverrideFact` is present, the execution becomes:
`Decision = f([... Common_Facts, Override_Fact], StateCapsule, Policy)`

The override is simply a new, high-priority input to the same deterministic function. It does not manually force an output; it provides a new input that deterministically leads to an expected outcome.

## Core Properties

- **Deterministic**: The same override fact will always produce the same outcome.
- **Auditable**: The `OverrideFact` is recorded in the Veritas audit trail like any other fact.
- **Costly**: Generating an `OverrideFact` incurs a significant reputational cost for the entity that issues it.
- **Policy-Gated**: The ability to create and honor `OverrideFacts` is strictly controlled by the active `POLICY`.
