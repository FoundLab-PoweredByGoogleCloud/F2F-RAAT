# Decision Semantics & Economic Binding

## Decision Types (Finite Set)

Valid decisions MUST be one of:

- `ALLOW`  
- `DENY`  
- `ALLOW_WITH_CONDITIONS`  
- `RATE_LIMIT`  
- `ESCROW_OR_HOLD`  
- `QUARANTINE`  
- `FLAG_ONLY`  
- `EMERGENCY_HALT` (only if explicitly enabled by policy + break-glass prerequisites).

## Validity Conditions

A decision SHALL be valid only if:

- It references immutable identifiers: `DecisionID`, `PolicyID`, `CapsuleHash`, `FactDigest`, `RuntimeVersion`.  
- It includes a bounded, schema-defined `Effects` structure (Map), each effect being policy-enumerated.  
- It is reproducible by re-executing the deterministic function over the same inputs.

## Economic Binding

- Economic binding MUST be implemented only via pre-committed rules in `POLICY` (e.g., slashing, holds, limits, pricing adjustments, collateral requirements).  
- The Burn Engine MUST only execute economically binding effects that are explicitly authorized by policy and validated by signatures/attestation.  
- No discretionary or subjective economic outcomes SHALL be permitted.
