# Interface Contracts
> **"The Standard Plug for Trust."**

This directory contains the machine-readable API specifications for integrating with F2F-RAaT.

## 📄 Specifications

| File | Type | Description |
| :--- | :--- | :--- |
| **[`f2f-raat-api.yaml`](f2f-raat-api.yaml)** | **OpenAPI 3.1** | The normative REST contract for the execution runtime. |
| **[`FACT_SCHEMAS/`](FACT_SCHEMAS/)** | **JSON Schema** | Standard conceptual models for common input facts (e.g. Velocity, Identity). |

## 🔌 Integration Patterns

F2F-RAaT is designed as a **Sidecar** or **Microservice**:

1.  **Your Service** constructs a `Fact`.
2.  **Your Service** retrieves the stored `StateCapsule` for that user (from your own DB/Blockchain).
3.  **Your Service** POSTs both to `/execute`.
4.  **F2F-RAaT** computes the decision and returns a `NewCapsule`.
5.  **Your Service** enforces the decision and saves the `NewCapsule`.

No database connection is required between your service and the Engine.

---
[⬅️ Back to Main Spec](../README.md)
