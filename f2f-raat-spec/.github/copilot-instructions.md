## F2F-RAAT Specification AI Coding Agent Instructions

This document provides guidance for AI coding agents working with the F2F-RAAT (From Fact to Feedback - Reputation as a Transaction) specification. The goal is to ensure that contributions are consistent with the project's architecture and design principles.

### "Big Picture" Architecture

The F2F-RAAT system is a deterministic, non-oracular reputational execution engine. It processes "Facts" to update a "State Capsule," which is then consumed by the "Burn Engine" to produce an economically binding "Decision." This entire process is auditable via the "Veritas" component.

The core data flow is as follows:
`Fact` -> `Spezzatura T²` -> `State Capsule` -> `Burn Engine` -> `Binding Decision` -> `Veritas Proof` -> `Reputational Feedback`

For a detailed understanding of the architecture, please refer to `WHITEPAPER/F2F-RAAT_WHITEPAPER_v1.md`. The Mermaid diagram in section 2 provides a clear visual representation of the data flow.

### Core Components and Key Files

The project is divided into several core components, each with a specific purpose. When working on a task, identify the relevant component and refer to the corresponding files.

- **`EXECUTION_CONTRACT/`**: Defines the formal constraints of the system. This is the source of truth for the runtime behavior.
- **`STATE_CAPSULE/`**: Contains the schema and specification for the `State Capsule`, which is a PII-free, cryptographically verifiable data structure that represents an entity's reputation.
- **`BURN_ENGINE/`**: Specifies the deterministic authorization and decision-making logic. It consumes `State Capsules` and `Facts` to produce binding actions.
- **`SPEZZATURA/`**: Describes the T² deterministic reputation model. This component is responsible for calculating reputation scores based on various vectors.
- **`VERITAS/`**: Outlines the immutable audit trail and cryptographic commitment mechanisms. All decisions made by the `Burn Engine` are recorded here.
- **`GOVERNANCE/`**: Contains rules for policy signing, break-glass protocols, and override facts.

### Project-Specific Conventions and Patterns

- **Determinism is key**: The entire system is designed to be deterministic. The same inputs should always produce the same outputs. Avoid introducing any randomness or non-deterministic behavior.
- **No PII**: The `State Capsule` and all other components are designed to be PII-free. Do not introduce any personally identifiable information into the system.
- **Immutability**: The `Veritas` component ensures that the audit trail is immutable. Do not attempt to modify or delete any existing records.
- **Formal Language**: The specification is written in a formal, precise language. When updating the documentation, maintain this style. Use MUST/SHOULD/MAY as defined in RFC 2119.

### Developer Workflows

This project is a specification, not a direct implementation. Therefore, there are no build or test commands. The primary workflow is to update the specification documents in a way that is consistent with the existing architecture and principles.

When making changes, consider the impact on all components of the system. For example, a change to the `State Capsule` schema may require updates to the `Burn Engine` and `Execution Contract`.

### Examples

- **Updating the `State Capsule` schema**: If you need to add a new field to the `State Capsule`, you must update `STATE_CAPSULE/capsule_schema.json` and `STATE_CAPSULE/capsule_spec.md`. You should also consider if this change affects the invariants defined in `EXECUTION_CONTRACT/invariants.md`.
- **Adding a new `Burn Engine` decision type**: To add a new decision type to the `Burn Engine`, you need to update `BURN_ENGINE/burn_engine_spec.md` and the `EXECUTION_CONTRACT/F2F-RAAT_EXECUTION_CONTRACT_v1.md` to reflect this change.

By following these guidelines, you can help ensure that the F2F-RAAT specification remains consistent, robust, and secure.
