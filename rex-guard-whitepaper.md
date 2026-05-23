# REX Guard: Cryptographic Separation and Deterministic Legal-State Governance for Immutable Auditability in Generative AI Systems

## Abstract

Regulated Generative AI systems require durable evidence of decisions, model behavior, policy context, and post-hoc accountability. However, immutable evidentiary storage creates a direct systems conflict with data-subject rights, particularly erasure or elimination requests under data protection regimes such as Brazil's LGPD. LGPD grants titular rights including elimination of certain personal data, while also permitting conservation for legal or regulatory obligations under specified conditions.[^1] In parallel, Brazilian financial regulation has moved toward more prescriptive cyber, cloud, traceability, and auditability requirements; Resolução BCB nº 538/2025 amends Resolução BCB nº 85/2021 concerning cybersecurity policy and requirements for processing, storage, and cloud services.[^2]

This paper defines **REX Guard**, an architectural pattern for reconciling immutable AI auditability with mutable privacy and retention obligations. The core mechanism is a strict separation of planes: an Immutable Evidence Ledger, a Mutable Key and Retention Plane, an Encrypted Sensitive Payload Store, and an Operational Analytics Plane. The central invariant is **Non-Coresidence**: raw personally identifiable information and cryptographic key material must never be committed to the immutable ledger. A pre-commit **Airlock Quarantine Protocol** enforces this invariant before WORM retention locks are applied. A deterministic legal state machine executes a versioned Legal Retention Precedence Matrix without deciding substantive law. Finally, crypto-shredding is formalized as a technical inaccessibility control, not as an unconditional claim of legal erasure.

---

## Table of Contents

1. [Introduction: The Erasure-Immutability Paradox](#1-introduction-the-erasure-immutability-paradox)
2. [System Architecture](#2-system-architecture)
3. [The Non-Coresidence Invariant](#3-the-non-coresidence-invariant)
4. [The Airlock Quarantine Protocol](#4-the-airlock-quarantine-protocol)
5. [Deterministic Retention State Machine](#5-deterministic-retention-state-machine)
6. [Cryptographic Topology and Claims](#6-cryptographic-topology-and-claims)
7. [End-to-End Decision Lifecycle](#7-end-to-end-decision-lifecycle)
8. [Failure Modes and Required Mitigations](#8-failure-modes-and-required-mitigations)
9. [Conclusion](#9-conclusion)
10. [References](#references)

---

## 1. Introduction: The Erasure-Immutability Paradox

Generative AI systems deployed in regulated financial, legal, or public-sector environments must preserve decision evidence sufficient for audit, supervision, dispute resolution, and internal control. A typical GenAI decision trace may include:

- user input or prompt fragments;
- retrieved context;
- model output;
- policy evaluation;
- tool calls;
- approval or rejection rationale;
- timestamps and transition metadata;
- controller-specific retention basis.

The difficulty is that such evidence may contain personal data. If personal data is written directly into WORM storage, immutable object locks, append-only logs, or regulatory archives, then later erasure or elimination requests cannot be technically satisfied without corrupting the evidentiary chain.

The paradox can be stated as two competing properties:

$$
\textbf{Audit Integrity:}\quad \forall e_i, e_j \in L,\ i < j \Rightarrow e_i \text{ remains immutable after } e_j
$$

$$
\textbf{Privacy Mutability:}\quad \exists r \in R_{\text{subject}} \text{ such that } \text{access}(PII_r) \rightarrow \bot
$$

Where \(L\) is the evidence ledger and \(R_{\text{subject}}\) denotes rights or retention events associated with a data subject. The conflict arises when:

$$
PII_r \in L \land L \text{ is immutable}
$$

In that case, erasure requires mutating \(L\), while audit integrity forbids mutating \(L\). REX Guard resolves the contradiction by ensuring that the immutable ledger contains integrity commitments, not the sensitive material itself.

This paper treats BCB 538 as part of a broader class of regulated cyber, cloud, traceability, and evidentiary controls, not as a GenAI-specific statute. The architecture is therefore framed as a technical governance pattern for systems subject to BCB 538-class auditability and resilience obligations. BCB's 2025 update strengthens cybersecurity-policy requirements in regulated environments, including controls connected to processing, storage, and cloud services.[^2]

---

## 2. System Architecture

REX Guard separates integrity, confidentiality, retention authority, and operational observability into four planes.

### 2.1 Plane I — Immutable Evidence Ledger

The **Immutable Evidence Ledger** is WORM or append-only storage. It stores only audit-safe commitments and governance metadata:

$$
L = \{B_1, B_2, \ldots, B_n\}
$$

Each bundle \(B_i\) may contain:

$$
B_i =
\left(
id_i,\
h(P_i),\
h(\pi_i),\
ref(T_i),\
Env_i,\
Cert^{airlock}_i,\
\sigma_i
\right)
$$

Where:

- \(id_i\) is a decision or evidence identifier;
- \(h(P_i)\) is a canonical hash of the sensitive payload;
- \(h(\pi_i)\) is a hash of the applicable policy snapshot;
- \(ref(T_i)\) references deterministic state transitions;
- \(Env_i\) is an evidence envelope without raw PII or key material;
- \(Cert^{airlock}_i\) is the signed Airlock Certificate;
- \(\sigma_i\) is the ledger-level signature or sealing proof.

The ledger is designed for verification, not disclosure. It must support later proof that a payload, policy, or transition existed in a specific canonical form without storing the payload itself.

### 2.2 Plane II — Mutable Key and Retention Plane

The **Mutable Key and Retention Plane** contains the lifecycle authority for encrypted sensitive data. It stores or controls:

- content-key state;
- retention state;
- TTL values;
- legal-hold status;
- erasure-request status;
- Legal Retention Precedence Matrix version;
- Retention Decision Records;
- destruction proofs.

For decision \(d\), the default topology is:

$$
K_d \leftarrow \textsf{KeyGen}(1^\lambda)
$$

Where \(K_d\) is a per-decision content key. The key plane is mutable because the system must support suspension, release, expiration, key destruction, and legal-hold changes after the immutable evidence has been sealed.

### 2.3 Plane III — Encrypted Sensitive Payload Store

The **Encrypted Sensitive Payload Store** contains ciphertext, not plaintext. For each decision \(d\):

$$
C_d = \textsf{Enc}_{K_d}(P_d; AAD_d)
$$

Where:

- \(P_d\) is the sensitive payload;
- \(K_d\) is the decision content key;
- \(AAD_d\) is associated data binding the ciphertext to the evidence context;
- \(C_d\) is stored outside the immutable ledger.

The plaintext payload \(P_d\) may include prompts, retrieved context, outputs, rationale, user attributes, and other sensitive fields. REX Guard requires that \(P_d\) never be committed to WORM storage.

### 2.4 Plane IV — Operational Analytics

The **Operational Analytics Plane** supports monitoring, aggregate statistics, SLO evaluation, incident triage, and governance dashboards. It is not an evidentiary source of truth and must not become a shadow retention bypass.

Formally, analytics outputs \(A\) must be derived under a minimization function:

$$
A = f_{\text{min}}(M)
$$

Where \(M\) is operational metadata and \(f_{\text{min}}\) excludes raw PII and content keys. If analytics artifacts become re-identifiable or decision-specific, they must be classified under the same retention and access controls as sensitive payloads.

---

## 3. The Non-Coresidence Invariant

The central safety rule is the **Non-Coresidence Invariant**.

Let:

$$
\mathcal{P} = \{x \mid x \text{ is raw PII or sensitive plaintext}\}
$$

$$
\mathcal{K} = \{x \mid x \text{ is plaintext, wrapped, derived, temporary, or recoverable key material}\}
$$

$$
\mathcal{W} = \{x \mid x \text{ is committed to WORM evidence storage}\}
$$

The strict invariant is:

$$
\boxed{
\mathcal{W} \cap (\mathcal{P} \cup \mathcal{K}) = \varnothing
}
$$

Equivalently, for every WORM bundle \(B_i\):

$$
\boxed{
B_i \subseteq
\mathcal{H}
\cup
\mathcal{\Pi}
\cup
\mathcal{R}
\cup
\mathcal{E}
\cup
\mathcal{C}
\cup
\mathcal{S}
}
$$

Where:

- \(\mathcal{H}\): canonical hashes;
- \(\mathcal{\Pi}\): policy snapshots or policy hashes;
- \(\mathcal{R}\): transition references;
- \(\mathcal{E}\): evidence envelopes;
- \(\mathcal{C}\): Airlock Certificates;
- \(\mathcal{S}\): signatures and sealing metadata.

And:

$$
\forall B_i \in L,\quad
B_i \cap \mathcal{P} = \varnothing
$$

$$
\forall B_i \in L,\quad
B_i \cap \mathcal{K} = \varnothing
$$

This is stronger than merely saying that PII and keys cannot appear together. It states that neither raw PII nor key material may appear in the immutable ledger at all.

### 3.1 Consequence

The ledger proves that an evidentiary payload existed and was governed by a specific policy and transition history. It does not itself contain the payload or the keys required to decrypt it.

Thus, after crypto-shredding, the ledger can still verify:

$$
h(P_d) = h(P_d')
$$

for a later-produced payload \(P_d'\), if lawfully available, while not enabling reconstruction of \(P_d\) from the ledger alone.

---

## 4. The Airlock Quarantine Protocol

The Airlock is a mandatory pre-commit protocol between capture and WORM sealing.

The admissible commit path is:

```text
CAPTURED
  -> QUARANTINE_PENDING_VALIDATION
  -> AIRLOCK_APPROVED
  -> SEALED_IMMUTABLE
```

No WORM retention lock may be applied before `AIRLOCK_APPROVED`.

### 4.1 Protocol Objective

The Airlock prevents the creation of irreversible cryptographic cemeteries: immutable stores containing either raw PII, key material, or both. Once such material is locked into WORM, neither erasure nor key destruction can reliably restore privacy control.

The Airlock therefore enforces a one-way safety gate:

$$
\textsf{CommitToWORM}(B_i)
\Rightarrow
\textsf{AirlockApproved}(B_i)
$$

And:

$$
\neg \textsf{AirlockApproved}(B_i)
\Rightarrow
\neg \textsf{RetentionLock}(B_i)
$$

### 4.2 Airlock Validation Gates

For candidate bundle \(B_i\), the Airlock evaluates the following gates.

#### Gate 1 — Canonicalization

The payload, policy snapshot, and transition record are canonicalized before hashing:

$$
h(P_i) = H(\textsf{Canon}(P_i))
$$

$$
h(\pi_i) = H(\textsf{Canon}(\pi_i))
$$

The plaintext payload \(P_i\) is not admitted to the WORM bundle.

#### Gate 2 — Non-Coresidence Validation

The bundle must satisfy:

$$
B_i \cap (\mathcal{P} \cup \mathcal{K}) = \varnothing
$$

If violated:

$$
state(B_i) \leftarrow \texttt{QUARANTINE\_REJECTED}
$$

No immutable commit occurs.

#### Gate 3 — Evidence Envelope Validation

The evidence envelope must contain only permitted fields:

$$
Env_i \subseteq
\mathcal{H}
\cup
\mathcal{\Pi}
\cup
\mathcal{R}
\cup
\mathcal{S}
$$

It may reference state transitions and policy versions, but must not embed payload plaintext or key material.

#### Gate 4 — Legal-State Binding

The evidence bundle must reference the applicable Legal Retention Precedence Matrix version:

$$
v_i = \textsf{MatrixVersion}(d_i, t_i)
$$

This binds the evidence to the governance rule set in force at the decision time.

#### Gate 5 — Airlock Certificate Generation

If all gates pass, the Airlock emits:

$$
Cert^{airlock}_i =
\textsf{Sign}_{sk_A}
\left(
id_i,\
h(B_i),\
v_i,\
t_i,\
\texttt{AIRLOCK\_APPROVED}
\right)
$$

The certificate is then included in the WORM bundle. The irreversible retention lock is applied only after this certificate exists.

### 4.3 Airlock Safety Property

The Airlock enforces:

$$
\boxed{
state(B_i)=\texttt{SEALED\_IMMUTABLE}
\Rightarrow
Cert^{airlock}_i \neq \bot
\land
B_i \cap (\mathcal{P} \cup \mathcal{K}) = \varnothing
}
$$

This is the minimum admissibility condition for immutable AI audit evidence.

---

## 5. Deterministic Retention State Machine

REX Guard does not decide material law. It executes a controller-governed Legal Retention Precedence Matrix.

Let:

$$
M_v
$$

be a versioned matrix approved by the relevant institutional controllers, such as DPO, Legal, and Compliance. The matrix encodes precedence among:

- regulatory retention;
- legal hold;
- data-subject erasure request;
- TTL expiration;
- manual arbitrage result;
- jurisdictional or business-purpose constraints.

### 5.1 State Space

Define the retention state space:

$$
S =
\{
\texttt{RETENTION\_ACTIVE},
\texttt{LEGAL\_HOLD\_ACTIVE},
\texttt{ERASURE\_REQUESTED},
\texttt{SHRED\_BLOCKED\_BY\_REGULATORY\_RETENTION},
\texttt{SHRED\_AUTHORIZED},
\texttt{CRYPTO\_SHREDDED}
\}
$$

Define the event space:

$$
E =
\{
\texttt{TTL\_EXPIRED},
\texttt{ERASURE\_REQUEST},
\texttt{LEGAL\_HOLD\_ASSERTED},
\texttt{LEGAL\_HOLD\_RELEASED},
\texttt{MATRIX\_UPDATED},
\texttt{MANUAL\_ARBITRAGE\_RECORDED}
\}
$$

Define the action space:

$$
A =
\{
\texttt{NOOP},
\texttt{SUSPEND\_SHRED},
\texttt{BLOCK\_SHRED},
\texttt{DESTROY\_CONTENT\_KEY},
\texttt{RECORD\_RDR}
\}
$$

The deterministic transition function is:

$$
\boxed{
\delta: S \times E \times M_v \rightarrow S
}
$$

The action function is:

$$
\boxed{
\alpha: S \times E \times M_v \rightarrow 2^A
}
$$

For identical inputs, the same state and actions must be produced:

$$
(s,e,M_v) = (s',e',M'_v)
\Rightarrow
\delta(s,e,M_v)=\delta(s',e',M'_v)
$$

$$
(s,e,M_v) = (s',e',M'_v)
\Rightarrow
\alpha(s,e,M_v)=\alpha(s',e',M'_v)
$$

### 5.2 Legal-Hold Dominance

A legal hold definitively suspends shredding.

$$
\forall s \in S,\quad
\delta(s,\texttt{LEGAL\_HOLD\_ASSERTED},M_v)
=
\texttt{LEGAL\_HOLD\_ACTIVE}
$$

$$
\forall e \in E,\quad
state=\texttt{LEGAL\_HOLD\_ACTIVE}
\Rightarrow
\texttt{DESTROY\_CONTENT\_KEY}
\notin
\alpha(state,e,M_v)
$$

This is a hard safety invariant:

$$
\boxed{
\textsf{LegalHold}(d,t)
\Rightarrow
\neg \textsf{DestroyKey}(K_d,t)
}
$$

### 5.3 Erasure Under Regulatory Retention

When an erasure request arrives during active regulatory retention, the system must not perform automatic shredding. It transitions to a blocked state pending matrix or manual arbitrage:

$$
\delta(
\texttt{RETENTION\_ACTIVE},
\texttt{ERASURE\_REQUEST},
M_v
)
=
\texttt{SHRED\_BLOCKED\_BY\_REGULATORY\_RETENTION}
$$

$$
\alpha(
\texttt{RETENTION\_ACTIVE},
\texttt{ERASURE\_REQUEST},
M_v
)
=
\{
\texttt{BLOCK\_SHRED},
\texttt{RECORD\_RDR}
\}
$$

Where `RDR` denotes a **Retention Decision Record**. The RDR records the matrix version, facts evaluated, state transition, and legal reasoning supplied by the controller-governed matrix. It does not independently decide legal correctness.

### 5.4 TTL Expiration Without Hold

If the TTL has expired, no legal hold exists, and the matrix authorizes destruction:

$$
\textsf{Allowed}_{M_v}(d,t)
=
\textsf{TTLExpired}(d,t)
\land
\neg \textsf{LegalHold}(d,t)
\land
\textsf{MatrixPermitsShred}(M_v,d,t)
$$

Then:

$$
\delta(
\texttt{RETENTION\_ACTIVE},
\texttt{TTL\_EXPIRED},
M_v
)
=
\texttt{SHRED\_AUTHORIZED}
$$

$$
\alpha(
\texttt{RETENTION\_ACTIVE},
\texttt{TTL\_EXPIRED},
M_v
)
=
\{
\texttt{DESTROY\_CONTENT\_KEY},
\texttt{RECORD\_RDR}
\}
$$

After confirmed key destruction:

$$
\delta(
\texttt{SHRED\_AUTHORIZED},
\texttt{MANUAL\_ARBITRAGE\_RECORDED},
M_v
)
=
\texttt{CRYPTO\_SHREDDED}
$$

The event label may represent either a controller-authorized final decision or a system-recorded confirmation event, depending on the institution's operating model.

### 5.5 State Machine Safety Invariants

#### Invariant 1 — No WORM Mutation

Retention transitions must not mutate prior WORM entries:

$$
\forall t_2 > t_1,\quad
L(t_2) = L(t_1) \Vert \Delta L
$$

No prior bundle \(B_i\) is rewritten.

#### Invariant 2 — Key Destruction Requires Authorization

$$
\textsf{DestroyKey}(K_d,t)
\Rightarrow
state(d,t^-)=\texttt{SHRED\_AUTHORIZED}
$$

#### Invariant 3 — Regulatory Block Requires RDR

$$
state(d,t)=\texttt{SHRED\_BLOCKED\_BY\_REGULATORY\_RETENTION}
\Rightarrow
\exists RDR_d
$$

#### Invariant 4 — Deterministic Matrix Execution

The system executes the matrix as code, not as discretionary runtime judgment:

$$
\delta(s,e,M_v)
\neq
\delta(s,e,M_{v+1})
$$

is permitted only when the matrix version changes. The version change itself must be recorded as evidence.

---

## 6. Cryptographic Topology and Claims

### 6.1 Per-Decision Content Keys

The default REX Guard topology uses one content key per decision:

$$
K_d \leftarrow \textsf{KeyGen}(1^\lambda)
$$

$$
C_d = \textsf{Enc}_{K_d}(P_d; AAD_d)
$$

This granularity minimizes collateral effects. Destroying \(K_d\) affects only the corresponding decision payload.

#### Advantages

- Fine-grained crypto-shredding.
- Lower risk of destroying unrelated subject or transaction records.
- Better compatibility with mixed-retention decisions.
- Cleaner mapping between Retention Decision Record and affected ciphertext.

#### Costs

- Higher key cardinality.
- Greater retention-plane indexing burden.
- More complex key lifecycle telemetry.
- Larger state-machine surface.

### 6.2 Per-Subject Key Alternative

A per-subject key topology can be represented as:

$$
K_s \leftarrow \textsf{KeyGen}(1^\lambda)
$$

$$
C_{d,s} = \textsf{Enc}_{K_s}(P_{d,s}; AAD_{d,s})
$$

This reduces key count but creates coupling among unrelated decisions. If one decision is subject to legal hold and another is eligible for erasure, a shared key creates conflict:

$$
\textsf{LegalHold}(d_1)
\land
\textsf{ShredEligible}(d_2)
\land
K(d_1)=K(d_2)
$$

This may force over-retention or over-deletion. Therefore, per-subject keys are inferior where retention bases vary at decision granularity.

### 6.3 Crypto-Shredding Definition

Crypto-shredding is defined as the irreversible destruction of the content key required to decrypt a ciphertext:

$$
\textsf{CryptoShred}(d)
:=
\textsf{Destroy}(K_d)
\land
\neg \textsf{Recoverable}(K_d)
$$

After crypto-shredding:

$$
\textsf{Dec}_{K_d}(C_d) \rightarrow \bot
$$

under the system's operational threat model.

The technical claim is:

$$
\boxed{
\textsf{CryptoShred}(d)
\Rightarrow
\textsf{PayloadInaccessible}_{\mathcal{A}}(C_d)
}
$$

Where \(\mathcal{A}\) is an adversary or actor without access to \(K_d\), assuming the encryption scheme remains secure and no plaintext replicas exist outside the governed planes.

### 6.4 Boundary of the Claim

REX Guard does **not** claim:

$$
\textsf{CryptoShred}(d) \equiv \textsf{LegalErasure}(d)
$$

The correct claim is narrower:

$$
\boxed{
\textsf{CryptoShred}(d)
\Rightarrow
\textsf{TechnicalInaccessibility}(P_d)
}
$$

Legal erasure depends on the governing legal regime, controller interpretation, retention basis, and regulatory obligations. Under LGPD, elimination rights interact with exceptions such as conservation for legal or regulatory obligations.[^1] Therefore, the system must record both:

$$
Proof^{destroy}_d
$$

and:

$$
RDR_d
$$

The destruction proof establishes that the content key was destroyed. The Retention Decision Record explains why destruction was allowed, blocked, suspended, or manually arbitrated.

### 6.5 Residual Evidence After Shredding

After crypto-shredding, the immutable ledger may still contain:

$$
h(P_d),\ h(\pi_d),\ ref(T_d),\ Cert^{airlock}_d,\ RDR_d
$$

This is intentional. The ledger preserves integrity and accountability while the payload becomes inaccessible through key destruction.

However, the architecture must recognize that hashes and metadata can sometimes remain sensitive depending on linkability, uniqueness, and auxiliary knowledge. Therefore, REX Guard does not treat all hashes as automatically non-personal in every legal context. It treats them as admissible ledger commitments only under the Non-Coresidence Invariant and the controller's classification policy.

---

## 7. End-to-End Decision Lifecycle

A complete REX Guard lifecycle is:

```text
1. CAPTURED
   Sensitive payload P_d is created.

2. QUARANTINE_PENDING_VALIDATION
   Candidate evidence bundle B_d is assembled without PII or key material.

3. AIRLOCK_APPROVED
   Airlock verifies Non-Coresidence and emits signed certificate.

4. SEALED_IMMUTABLE
   B_d is committed to WORM ledger.

5. RETENTION_ACTIVE
   Mutable retention plane manages TTL, matrix version, and hold status.

6. ERASURE_REQUESTED or TTL_EXPIRED
   Event enters deterministic retention state machine.

7. LEGAL_HOLD_ACTIVE
   Any shredding is suspended.

8. SHRED_BLOCKED_BY_REGULATORY_RETENTION
   Erasure request is blocked pending matrix/manual arbitrage.

9. SHRED_AUTHORIZED
   Matrix/manual process authorizes key destruction.

10. CRYPTO_SHREDDED
    Content key is destroyed; proof and RDR are recorded.
```

The key property is that immutable integrity evidence and mutable privacy controls remain orthogonal.

---

## 8. Failure Modes and Required Mitigations

### 8.1 WORM PII Contamination

**Failure:** Raw PII enters the immutable ledger.

**Impact:** Erasure becomes technically infeasible without corrupting evidence.

**Mitigation:** Mandatory Airlock validation before retention lock.

$$
\neg \textsf{AirlockApproved}(B_i)
\Rightarrow
\neg \textsf{CommitToWORM}(B_i)
$$

### 8.2 Key Material Leakage Into Evidence

**Failure:** Plaintext, wrapped, derived, or temporary key material is written to WORM.

**Impact:** Crypto-shredding becomes invalid because the key remains recoverable.

**Mitigation:** Strict key exclusion from WORM:

$$
\mathcal{W} \cap \mathcal{K} = \varnothing
$$

### 8.3 Legal Hold Race Condition

**Failure:** A key is destroyed after a legal hold is asserted but before the retention plane processes the hold.

**Impact:** Evidence becomes inaccessible despite preservation obligation.

**Mitigation:** Legal-hold dominance and serialized state transitions.

$$
\textsf{LegalHold}(d,t)
\Rightarrow
\texttt{DESTROY\_CONTENT\_KEY}
\notin
\alpha(state,e,M_v)
$$

### 8.4 Matrix Drift

**Failure:** Retention decisions are made under unversioned or silently modified rules.

**Impact:** Non-reproducible legal-state transitions.

**Mitigation:** Every transition binds to \(M_v\), and matrix updates become auditable events.

$$
Transition_d =
(s,e,M_v,s',a,t,\sigma)
$$

### 8.5 Analytics Re-Identification

**Failure:** Operational analytics recreate subject-specific traces outside governed retention controls.

**Impact:** Shadow retention plane.

**Mitigation:** Analytics must be minimized, classified, and excluded from raw PII/key custody.

---

## 9. Conclusion

REX Guard resolves the erasure-immutability paradox by refusing to place privacy-bearing material inside immutable evidence. The design separates:

$$
\text{Integrity} \neq \text{Plaintext Retention}
$$

$$
\text{Auditability} \neq \text{Permanent PII Storage}
$$

The architecture's core contribution is not a new encryption primitive, but a governance-safe composition of four mechanisms:

1. immutable evidence without raw PII or key material;
2. mutable key and retention control;
3. encrypted sensitive payload storage;
4. deterministic legal-state execution through a versioned matrix.

The Airlock prevents irreversible contamination before WORM locking. The state machine prevents discretionary or inconsistent retention behavior. Crypto-shredding provides technical inaccessibility while preserving an honest boundary: it is a technical control, not a universal legal-erasure theorem.

For regulated GenAI deployments, this pattern enables durable evidence, reproducible governance, and controlled privacy mutability without collapsing audit integrity into permanent sensitive-data retention.

---

## References

[^1]: Lei Geral de Proteção de Dados Pessoais — Article 18, data subject rights and elimination rights. <https://lgpd-brasil.info/capitulo_03/artigo_18>

[^2]: Banco Central do Brasil — Resolução BCB nº 538/2025, amending Resolução BCB nº 85/2021 on cybersecurity policy and requirements for processing, storage, and cloud services. <https://www.bcb.gov.br/estabilidadefinanceira/exibenormativo?numero=538&tipo=Resolu%C3%A7%C3%A3o+BCB>
