# Mitigation Patterns (Overview)

## Governance & Policy Mitigations
- **Strong Policy Governance**: Mitigates policy capture and authority creep through multi-sig requirements, HSM-backed keys, and mandatory external review for major changes.
- **Override Cost**: Mitigates break-glass abuse by ensuring every `OverrideFact` carries a significant, auditable reputational cost and is subject to multi-sig authorization.
- **Fixed Capability Sets**: Mitigates the risk of economic effects being used as weapons by versioning the `POLICY` and ensuring the set of possible effects is fixed and auditable.

## Technical & Cryptographic Mitigations
- **Anti-Rollback**: Mitigates rollback and anti-slashing evasion via monotonic counters and anchoring Veritas proofs in a WORM (Write-Once, Read-Many) ledger.
- **Fact Attestation**: Mitigates fact forgery and evidence laundering through dual-pipeline fact generation, requiring independent attestation.
- **Constant-Time Patterns**: Mitigates side-channel and timing analysis by using constant-time patterns in cryptographic operations and isolating sensitive computations where feasible.
- **Ex-Ante Simulation**: Mitigates the risk of a misconfigured P(x) model by requiring ex-ante simulation with test vectors before policy deployment.

## Procedural Mitigations
- **Dispute Protocol**: Provides a formal mechanism for contesting facts, reducing the impact of potentially malicious or incorrect inputs.
- **Schema & Class Definitions**: Mitigates ambiguity in the fact generation process by requiring strict schemas and defined classes for all `Fact` types.
