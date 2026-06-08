# Cluster Profile PhenoCard Schema

This repository contains the formal JSON Schema definition for the **Clinical Cluster Analysis Profile Card** (`federated-profile-card/2026.3.0`).

This schema defines a standardized container for hierarchical cluster profiles generated from federated clinical datasets. It bridges the gap between raw clustering outputs and downstream biomedical data tools by enforcing interoperability with **OMOP CDM, DCAT, PAV, and MINIMAR** standards. It is intended to be packaged alongside clinical datasets as part of an **RO-Crate**.

## Repository Structure

* `schema.json` — The core JSON Schema definition (Draft 2020-12).
* `examples/` — Valid and invalid payload examples for testing.
* `README.md` — This file.

---

## Data Architecture

The schema enforces a strict 4-layer structure to capture everything from semantic context down to individual feature constraints:

| Layer | Key | Purpose |
| --- | --- | --- |
| **L0** | `@context` | JSON-LD semantic bindings |
| **L1** | `global_metadata`, `dataset_metadata`, `provenance` | Study-level descriptors & reproducibility tracking |
| **L2** | `profiles` | Clinical cohort profiles (keyed by `profile_id`) |
| **L3** | `clusters` | Statistical and clinical sub-populations within each profile |

---

## Quick Start & Validation

To validate your generated cluster profile JSON files against this schema, you can use any standard JSON Schema validator.

### Python Example

```python
import json
from jsonschema import validate, ValidationError

# Load the schema
with open("schema.json", "r") as f:
    schema = json.load(f)

# Load your profile card
with open("my_cluster_profile.json", "r") as f:
    instance = json.load(f)

try:
    validate(instance=instance, schema=schema)
    print("Validation successful! File matches federated-profile-card/2026.3.0.")
except ValidationError as e:
    print(f"Validation failed: {e.message}")

```

---

## Schema Specification Reference

### 1. Root Object Constraints

* **Type:** `object`
* **Additional Properties:** `false` (Strict structural validation)

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `@context` | `Context` | ✅ | JSON-LD semantic context |
| `global_metadata` | `GlobalMetadata` | ✅ | Authorship, license, and publication narrative |
| `dataset_metadata` | `DatasetMetadata` | ✅ | Source dataset descriptors (DCAT format) |
| `provenance` | `Provenance` | ✅ | Computational run fingerprint for pipeline auditing |
| `profiles` | `map<string, Profile>` | ✅ | Dictionary of cohort profiles. Enforces a minimum of 1 entry. |

---

### 2. Core Components

#### Context (`Context`)

Handles JSON-LD semantic vocabularies. Allows node-level extensions (`additionalProperties: true`).

* Fixed mappings include: `@vocab`, `dcat`, `omop`, `MINIMAR`, and `pav`.

#### Global Metadata (`GlobalMetadata`)

Maps to **PAV Agent**. Tracks the `author` (Name, Affiliation, Optional ORCID), `license` (URI), `doi` (Full URI format), and an optional `semantic_summary` (Max 256 chars).

#### Dataset Metadata (`DatasetMetadata`)

Maps to **DCAT Dataset**. Acts as file pointers for the RO-Crate packager.

* **Privacy Guard:** Enforces `total_study_n` to be an integer $\ge 32$ to prevent re-identification in small cohorts.
* Requires either an external `aggregate_profile_reference` or an `inline_aggregate_profile` (demographic baseline characteristics array).

#### Provenance (`Provenance`)

Maps to **PAV Provenance**. Implements strict validation tracking:

* Timestamps: `execution_start_time` and `execution_end_time` (ISO 8601 format).
* Code auditing: `git_commit` (Regex pattern accepts 7-char short or 40-char full SHA-1).
* Data integrity: `data_version_hash` (Regex pattern accepts SHA-256 string).

#### Cohort Profiles (`Profile`)

Represents a labeled subpopulation. Contains clustering algorithmic details, clinical descriptions, and a qualitative `clustering_outcome` enum (`distinct_and_pure`, `distinct_but_mixed`, `indistinct_but_pure`, `indistinct_and_mixed`).

* **Security Note:** `clinician_notes` are limited to 1024 characters. Downstream renderers **must** apply XSS sanitization before injecting this field into the DOM.

#### Phenotype Signatures & Clusters (`Cluster`)

Defines the sub-population. Features are evaluated via conditional logic:

* Enforces boundaries using `AnchorOperator` rules (`>`, `<`, `>=`, `<=`, `==`, `BETWEEN`, `EXISTS`).
* Constraints are evaluated conditionally via JSON Schema `if-then` blocks triggered by the presence of an `anchor_operator`.
* **Privacy Guard:** The minimum population count (`n_count`) for any single reported cluster is explicitly locked to $\ge 10$.

---

## Standard Compliance Mappings

This schema normalizes clinical modeling output into the following open standards:

* **Semantic Layer:** JSON-LD 1.1 Compliance
* **Provenance:** PAV (Provenance, Authoring and Versioning) using `pav:authoredBy`, `pav:createdOn`, etc.
* **Dataset Discovery:** DCAT (Data Catalog Vocabulary) for metadata discovery.
* **Clinical Vocabulary:** OMOP CDM / Athena concept mappings for cross-institutional alignment.
* **Model Reporting:** MINIMAR (Minimum Information for Medical AI Reporting) compatibility for algorithmic configurations.

---

## License

Distributed under the metadata usage terms specified in the root execution layer (`global_metadata.license`).