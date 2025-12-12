# State Capsule Specification

The State Capsule is a PII-free, non–Turing-complete artifact representing reputational state.

It MAY contain:

- Reputational primitives derived from Spezzatura and Sigmoid.  
- Flags and XAI tokens.  
- Cryptographic proofs / commitments.  
- Constraints and version metadata.

It MUST NOT contain:

- PII, raw documents, free text, embeddings, or arbitrary payloads.  
- Any executable code.
