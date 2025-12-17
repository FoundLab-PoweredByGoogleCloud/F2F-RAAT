# Fact Schemas
> **"Standardized Inputs for the Reputation Engine."**

This directory contains strict JSON Schemas (Draft 2020-12) for common `Fact` types used in the F2F-RAaT ecosystem.

These schemas define the expected structure of the `dimensions` object for specific `fact_type`s.

## 📚 Registry

| Type | Schema File | Description |
| :--- | :--- | :--- |
| **`GEO_VELOCITY_VIOLATION`** | **[`geo_velocity.json`](geo_velocity.json)** | Measures impossible travel speed between two coordinates. |
| **`IDENTITY_CHALLENGE_SUCCESS`** | **[`identity_challenge.json`](identity_challenge.json)** | Records a successful user verification event (e.g. Biometrics). |

## 🛠️ Usage

When submitting a `Fact` to the API, validaate your `dimensions` payload against these schemas to ensure the Spezzatura Engine can correctly compute the vectors.
