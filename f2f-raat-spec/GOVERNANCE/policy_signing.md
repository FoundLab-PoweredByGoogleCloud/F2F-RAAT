# Policy Signing & Publication

- Policies MUST be signed using institutional keys (HSM-backed).  
- Publication SHOULD include a public `PolicyID` and checksum.  
- Different environments (dev/test/prod) MUST NOT reuse the same keys.
