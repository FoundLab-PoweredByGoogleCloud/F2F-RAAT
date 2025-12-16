# Spezzatura T² — Deterministic Reputation Model Specification

**Version:** 1.0.1-fix
**Component:** F2F-RAaT Reputation Engine
**Status:** APPROVED (ARTREV 16/12/2025)
**Author:** FoundLab Architecture Team

---

## 1. Overview

Spezzatura T² is the core deterministic reputation model within the **F2F-RAaT** execution engine. Unlike traditional "black box" AI scores, Spezzatura operates as a **transparent, structured Multi-Dimensional Vector State Machine**.

It captures reputational state without statistical inference or predictive modeling, ensuring that identical input facts and policies always produce bit-identical outputs (Determinism).

This model is designed to work in tandem with the **Sigmoid P(x)** engine, which handles acute/temporary reactivity, while Spezzatura T² handles structural/long-term reputation.

---

## 2. State Representation

The reputational state at time $t$ is represented as a normalized 6-dimensional vector:

$$
\mathbf{R}_t = (C_t, A_t, T_t, U_t, R_t, \hat{A}_t) \in [0, 1]^6
$$

### 2.1 Dimensions (The T² Vector)

| Dimension | Symbol | Description | Optimization Goal | Legacy Mapping (v0.9) |
| :--- | :---: | :--- | :---: | :--- |
| **Compliance** | $C$ | Adherence to regulatory rules and internal policy. | Maximize (↑) | — |
| **Activity** | $A$ | Volume and frequency of legitimate operations. | Maximize (↑) | — |
| **Trustworthiness** | $T$ | Historical consistency, longevity, and reliability. | Maximize (↑) | *Time & Regularity* |
| **Usage** | $U$ | Resource utilization patterns and efficiency. | Maximize (↑) | *Uniqueness* |
| **Risk** | $R$ | Exposure to illicit, anomalous, or toxic behavior. | Minimize (↓) | *Relationships* |
| **Intent (Ânimo)** | $\hat{A}$ | Inferred behavioral intent (coherent vs. erratic). | Maximize (↑) | — |

### 2.2 The Ideal State

The target state for a perfectly trusted entity is defined as:

$$
\mathbf{R}_{\text{ideal}} = (1.0, 1.0, 1.0, 1.0, 0.0, 1.0)
$$

Any deviation from this state represents "Reputational Distance".

---

## 3. Update Logic (The Transition Function)

The state transition is triggered by an incoming, typed **Fact** ($e_t$). The update logic follows a three-step deterministic pipeline.

### Step 1: First-Order Impact (Linear)
The fact carries a defined impact vector ($\Delta \mathbf{e}_t$) based on the active Policy.

$$
\mathbf{R}_{t+1}' = \mathbf{R}_t + \Delta \mathbf{e}_t
$$

### Step 2: Second-Order Adjustment (Quadratic Penalty)
To model the principle that *"trust is hard to gain but easy to lose"*, we apply a quadratic penalty proportional to the entity's distance from the Ideal State.

$$
\mathbf{R}_{t+1}'' = \mathbf{R}_{t+1}' - \alpha \cdot \|\mathbf{R}_{t+1}' - \mathbf{R}_{\text{ideal}}\|^2_2 \cdot \mathbf{w}_{\text{penalty}}(e_t)
$$

Where:
* $\alpha$: The global penalty coefficient (defined in Policy).
* $\|\cdot\|^2_2$: The squared Euclidean distance to the ideal state.
* $\mathbf{w}_{\text{penalty}}$: A directional weight vector ensuring penalties apply to the correct dimensions.

### Step 3: Normalization (Clipping)
The final state is clipped to ensure all dimensions remain within valid bounds $[0, 1]$.

$$
\mathbf{R}_{t+1} = \text{clip}_{[0,1]} \left( \mathbf{R}_{t+1}'' \right)
$$

---

## 4. Execution Flow Diagrams

### 4.1 State Transition Pipeline

mermaid
flowchart TD
    subgraph Input
      S0[Current State R_t]
      F[Fact + Policy Delta]
    end

    S0 & F --> L1[Step 1: First-Order Linear Update]
    L1 --> L2[Intermediate State R']
    
    L2 --> C1{Calc Distance to Ideal}
    C1 -->|Euclidean Dist²| P1[Step 2: Calculate Quadratic Penalty]
    P1 --> A1[Apply Penalty to R']
    
    A1 --> N1[Step 3: Clip & Normalize]
    N1 --> OUT[New State R_t+1]

    style OUT fill:#d4edda,stroke:#28a745,stroke-width:2px


4.2 Interaction with Sigmoid P(x)
Spezzatura runs in parallel with the high-reactivity Sigmoid model.
flowchart LR
    FACT((Fact Event)) --> SPEZ[Spezzatura T² Engine]
    FACT --> SIG[Sigmoid P x Engine]

    SPEZ -->|Update| R[Structural State R]
    SIG -->|Update| P[Reactivity Score P]

    R & P --> DEC[Burn Engine Decision]
    
    DEC -->|Threshold Met| ACTION[Block / Verify]
    DEC -->|Threshold OK| ALLOW[Allow]

    style SPEZ fill:#e1f5fe
    style SIG fill:#fff3cd


5. Audit Example (Traceability)
This section provides a normative example for auditors to verify the implementation.
Scenario: A trusted entity commits a POLICY_VIOLATION_HIGH.
0. Initial State:
1. Policy Parameters:
Global Alpha (\alpha): 0.05
Fact Impact (\Delta e): [-0.20, -0.10, -0.25, -0.05, +0.30, -0.15]
Penalty Weights (w): [0.2, 0.3, 0.4, 0.2, 0.6, 0.3]
2. Step 1 (Linear Update):
3. Step 2 (Quadratic Penalty):
Distance to Ideal: (1-0.3)^2 + (1-0.4)^2 + ... \approx 2.055
Scalar Penalty Factor: 0.05 \times 2.055 \approx 0.10275
Applied Penalty Vector: 0.10275 \times w \approx [0.02, 0.03, 0.04, 0.02, 0.06, 0.03]
4. Final State: Note the compounding loss in Trustworthiness (T) and increase in Risk (R) beyond the linear impact. The entity has been penalized non-linearly for deviating from the ideal.
6. System Invariants
Bitwise Determinism: The function update(State, Fact, Policy) MUST return the exact same floating-point result on any architecture (requires rigorous IEEE 754 compliance in the runtime).
Bounded State: No dimension of the vector may ever exceed the range [0.0, 1.0].
Monotonic Decay: In the absence of positive "Recovery Facts", the Time decay (handled by Sigmoid) does not affect the structural Spezzatura score directly, ensuring stability.
Veritas Binding: Every state transition generates a unique hash used in the Veritas audit chain.
<!-- end list -->

