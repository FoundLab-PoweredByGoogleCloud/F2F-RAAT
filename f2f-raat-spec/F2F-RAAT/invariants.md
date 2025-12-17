# Execution Invariants

## Determinism & Boundedness

- The runtime MUST implement:

```text
Decision = f(FACT, STATE_CAPSULE, POLICY, RUNTIME_VERSION)
```

- Output MUST be bitwise-deterministic for identical inputs.  
- Execution MUST be bounded-time and bounded-memory for all valid inputs.  
- Output MUST be drawn from a finite, versioned decision/effect set.

## Zero-persistence & Non-leakage

- The runtime MUST NOT persist raw inputs, intermediate states, or side artifacts outside allowed State Capsule + Veritas commitments.  
- Any transient buffers MUST be zeroized at end-of-execution.  
- The runtime MUST NOT emit outputs whose entropy or length can encode sensitive data (“exfiltration-by-output”).

## Constant-time & Fail-Closed

- The Burn Engine SHOULD follow constant-time patterns within defined bounds and MUST avoid secret-dependent branching where feasible.  
- On any validation failure, attestation failure, capsule invalidity, or policy mismatch, the engine MUST default to the most restrictive safe outcome defined by policy.  

## Policy Integrity

- `POLICY` MUST be versioned, signed, and referenced by immutable `PolicyID`.  
- The runtime MUST reject any policy not meeting signature/attestation requirements.
