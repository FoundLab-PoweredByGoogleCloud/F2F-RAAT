# Policy Governance

- Policy definitions MUST be controlled under strict separation-of-duties.  
- The entity proposing policy, signing policy, and publishing policy SHOULD be distinct roles.  
- Policy updates SHOULD go through:

1. Capability diffing (no unintended escalation).  
2. External or independent review for MAJOR changes.  
3. Regeneration of `PolicyID` and signatures.  

- Major additions to the effect set MUST require a MAJOR version bump and external audit.
