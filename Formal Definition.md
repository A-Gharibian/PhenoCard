# Formal Definition
## Cluster PhenoCard

**URI:** `https://gitlab.vsb.cz/gha0011/phenocard/-/blob/main/formal_definition.md?ref_type=heads`
**JSON Schema Draft:** 2020-12  
**Status:** Under Development

---

## 1. Overview

This schema defines a container for hierarchical cluster profiles generated from a single clinical dataset. 
It is designed to be interoperable and is intended to be packaged as part of an RO-Crate.

The document is structured in four layers:

| Layer | Key | Purpose |
|-------|-----|---------|
| L0 | `@context` | JSON-LD semantic bindings |
| L1 | `global_metadata`, `dataset_metadata`, `provenance` | Study-level descriptors |
| L2 | `profiles` | One or more clinical cohort profiles |
| L3 | `clusters` | Sub-populations within each profile |

---

## 2. Root Object

**Type:** `object` — `additionalProperties: false`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `@context` | `Context` | ✅ | JSON-LD semantic context |
| `global_metadata` | `GlobalMetadata` | ✅ | Authorship, license, semantic summary |
| `dataset_metadata` | `DatasetMetadata` | ✅ | Source dataset descriptors |
| `provenance` | `Provenance` | ✅ | Computational run fingerprint |
| `profiles` | `map<string, Profile>` | ✅ | Dictionary of profiles, keyed by `profile_id`. Min 1 entry. |

---

## 3. Type Definitions

---

### 3.1 `Context`
JSON-LD semantic context. `additionalProperties: true` to allow node-level extension.

| Field | Const Value |
|-------|-------------|
| `@vocab` | `https://gitlab.vsb.cz/gha0011/phenocard/terms/` |
| `dcat` | `https://www.w3.org/ns/dcat#` |
| `omop` | `https://athena.ohdsi.org/concept/` |
| `MINIMAR` | `https://www.equator-network.org/reporting-guidelines/minimar` |
| `pav` | `http://purl.org/pav/` |
| `dataset_metadata` | `@nest` |
| `global_metadata` | `@nest` |
| `publisher` | `dcat:publisher` |
| `dataset_title` | `dcat:title` |
| `license` | `dcat:license` |

---

### 3.2 `GlobalMetadata`
Shared publication metadata. Maps to **PAV Agent**.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `author` | `Author` | ✅ | Researcher identity and affiliation |
| `license` | `string (uri)` | ✅ | DCAT usage license URI |
| `doi` | `string \| null` | ❌ | DOI of the associated publication. Must be a full URI: `https://doi.org/10.{registrant}/{suffix}` |
| `semantic_summary` | `string` | ❌ | Keywords or short narrative summary of the dataset. Max 256 characters. |

#### 3.2.1 `Author`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | ✅ | Full name of the author, e.g. `Dr. Jane Doe` (`pav:authoredBy`) |
| `affiliation` | `string` | ✅ | Institutional affiliation, e.g. `European Federated Health Node` |
| `orcid` | `string \| null` | ❌ | ORCID URI, e.g. `https://orcid.org/0000-0000-0000-0000` (`pav:authoredBy` IRI form) |

---

### 3.3 `DatasetMetadata`
Source dataset descriptors. Acts as file pointers for the RO-Crate packager. Maps to **DCAT Dataset**.

At least one of `inline_aggregate_profile` or `aggregate_profile_reference` must be present.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `dataset_title` | `string` (minLength: 4) | ✅ | — | DCAT title |
| `total_study_n` | `integer` (min: 32) | ✅ | — | Absolute baseline population size. Minimum enforced for privacy protection. |
| `dqa_reference` | `string (uri-reference)` | ❌ | `results_DQA.json` | Pointer to an external data quality assessment results file |
| `aggregate_profile_reference` | `string (uri-reference)` | ❌ | — | Pointer to an external aggregate demographic profile file. Required if `inline_aggregate_profile` is omitted. |
| `inline_aggregate_profile` | `array<AggregateStatistic>` (minItems: 1) | ❌ | — | Embedded baseline characteristics table. Required if `aggregate_profile_reference` is omitted. |

---

### 3.4 `AggregateStatistic`
A single row of a baseline characteristics table.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `variable_role` | `string` | ✅ | e.g. `Demographic`, `Confounder`, `Target_Label` |
| `variable_name` | `string` (minLength: 1) | ✅ | Human-readable variable name |
| `concept_id` | `integer \| null` | ❌ | Concept identifier from the node's reference vocabulary |
| `data_type` | `"Continuous" \| "Categorical"` | ✅ | Variable type |
| `category_value` | `string` | ❌ | For categorical rows, the specific category label |
| `metric_mean_or_count` | `number` | ✅ | Mean (continuous) or count (categorical) |
| `metric_sd_or_proportion` | `number` (min: 0) | ✅ | SD (continuous) or proportion (categorical) |

---

### 3.5 `Provenance`
Timestamps, pipeline identifier, and code version for the run that produced this file. Maps to **PAV Provenance**.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `execution_start_time` | `string (date-time)` | ✅ | ISO 8601 timestamp when the pipeline run started (`pav:createdOn`) |
| `execution_end_time` | `string (date-time)` | ✅ | ISO 8601 timestamp when the pipeline run completed (`pav:lastUpdateOn`) |
| `pipeline_run_id` | `string` | ❌ | Unique identifier for this pipeline execution instance (`pav:version`) |
| `git_commit` | `string` | ❌ | Git commit SHA identifying the exact pipeline code version. Accepts short (7-char) and full (40-char) SHA-1. Pattern: `^[a-f0-9]{7,40}$` |
| `data_version_hash` | `string` | ❌ | SHA-256 hash of the input dataset for local reproducibility verification only. Not dereferenceable. Pattern: `^[a-f0-9]{64}$` |

---

### 3.6 `Profile`
A clinical cohort profile representing a labeled subpopulation and its clustering results.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `profile_id` | `string` (minLength: 1) | ✅ | Must match the parent map key |
| `base_cohort_id` | `integer \| null` | ❌ | Upstream cohort ID, e.g. an OMOP ATLAS Cohort ID |
| `base_cohort_reference` | `string (uri-reference)` | ✅ | Pointer to the cohort definition file, e.g. a CapR/CIRCE JSON |
| `algorithmic_config` | `AlgorithmicConfig` | ✅ | Clustering algorithm configuration |
| `label_relationship` | `LabelRelationship` | ✅ | Clinical label and concept mapping |
| `clustering_outcome` | `ClusteringOutcome` | ✅ | Qualitative assessment of cluster quality |
| `clusters` | `map<string, Cluster>` | ✅ | Dictionary of clusters, keyed by `cluster_id`. Min 1 entry. |
| `avg_condition_occurrence_days` | `number \| null` | ✅ | Average days of condition occurrence for this profile's cohort |
| `clinician_notes` | `string` | ✅ | Clinician evaluation for the profile. Max 1024 characters. Downstream renderers must apply XSS sanitization before injecting into the DOM. |

#### 3.6.1 `ClusteringOutcome` Enum

| Value | Meaning |
|-------|---------|
| `distinct_and_pure` | Clusters are well-separated and class-homogeneous |
| `distinct_but_mixed` | Clusters are well-separated but contain mixed classes |
| `indistinct_but_pure` | Clusters overlap but are class-homogeneous |
| `indistinct_and_mixed` | Clusters overlap and contain mixed classes |

---

### 3.7 `AlgorithmicConfig`
The clustering algorithm configuration applied to generate the profile. Maps to **MINIMAR**.

| Field | Type | Required | Default | Description |
|-------|------|----------|---------|-------------|
| `method_family` | `string` | ✅ | — | Algorithm family, e.g. `Distance-based Clustering` |
| `data_preprocessing` | `DataPreprocessing` | ✅ | — | Missingness and scaling strategy |
| `minimar_reference` | `string (uri-reference) \| null` | ❌ | `minimar_parameters.json` | Pointer to supplementary MINIMAR file |
| `base_model` | `string` | ❌ | — | Specific algorithm, e.g. `K-Means` |
| `confidence_threshold` | `number [0, 1]` | ❌ | — | Minimum probability score required to assign a patient to a specific cluster |
| `implementation_version` | `string` | ❌ | — | e.g. `scikit-learn-1.3.0` |
| `random_seed` | `integer \| null` | ❌ | — | Random seed for reproducibility |

#### 3.7.1 `DataPreprocessing`

| Field | Type | Required | Allowed Values |
|-------|------|----------|----------------|
| `missing_data_strategy` | `string` | ✅ | `complete_case_analysis`, `mean_imputation`, `median_imputation`, `knn_imputation`, `multiple_imputation`, `none` |
| `scaling_method` | `string` | ✅ | `standard_scaler_z_score`, `min_max_scaler`, `robust_scaler`, `none` |

---

### 3.8 `LabelRelationship`
The clinical label assigned to this profile and its concept mapping.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `assigned_class` | `string` (minLength: 1) | ✅ | Human-readable class label |
| `assigned_concept_id` | `integer` (min: 1) | ❌ | Concept identifier from the node's reference vocabulary, e.g. an ATHENA concept ID for OMOP networks |

---

### 3.9 `Cluster`
A single sub-population within a profile.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `cluster_id` | `string` (minLength: 1) | ✅ | Must match the parent map key |
| `phenotype_signature` | `PhenotypeSignature` | ✅ | The set of clinical variables and constraints that characterise this cluster |
| `covariance_matrix` | `array<CovariancePair>` | ❌ | Pairwise feature covariances. Symmetric convention assumed; upper triangle recommended. |
| `population` | `Population` | ✅ | Size and fraction descriptors |
| `quality_metrics` | `QualityMetrics` | ✅ | Validation metric collection |

---

### 3.10 `PhenotypeSignature`
The set of clinical variables and constraints that characterise a cluster.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `features` | `array<VariableConstraint>` (minItems: 1, uniqueItems) | ✅ | Defining feature rules |
| `integrity_hash` | `string` | ❌ | SHA-256 integrity hash of the feature set. Pattern: `^[a-f0-9]{16}$`. See specification for input canonicalisation. |
| `normalization_basis` | `"raw" \| "z_score" \| "min_max" \| "robust"` | ❌ | Scaling applied to feature anchor values and distributions. Default: `raw` |

---

### 3.11 `VariableConstraint`
A single clinical variable with its inclusion criteria and optional distributional summary.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `variable_name` | `string` (minLength: 1) | ✅ | Human-readable variable name |
| `concept_id` | `integer \| null` | ✅ | Concept identifier from the node's reference vocabulary |
| `unit_concept_id` | `integer \| null` | ❌ | Concept identifier for the unit of measurement, e.g. UCUM or OMOP unit concept |
| `anchor_operator` | `AnchorOperator` | ✅ | Inclusion comparison operator |
| `anchor_value` | `number \| string \| array \| boolean \| null` | ✅ | Threshold value(s). Type constrained by operator — see below. |
| `dist_mean` | `number` | ✅ | Cluster centre for this variable |
| `dist_sd` | `number` (min: 0) | ✅ | Cluster spread for this variable |
| `covariate_importance` | `number` | ❌ | Feature importance score from the upstream model |

#### 3.11.1 `AnchorOperator` — Conditional Constraints

| Operator | `anchor_value` required? | `anchor_value` type |
|----------|--------------------------|---------------------|
| `>`, `<`, `>=`, `<=`, `==` | ✅ | `number \| string` |
| `BETWEEN` | ✅ | `array` of exactly 2 `[number \| string]` |
| `EXISTS` | ❌ | `boolean \| null` |

> **Note:** Constraints are enforced via `allOf` / `if-then` in JSON Schema. Each `if` block includes `required: ["anchor_operator"]` to prevent evaluation when the field is absent.

---

### 3.12 `CovariancePair`
The statistical relationship between two features.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `feature_1_concept_id` | `integer \| null` | ✅ | First feature concept identifier |
| `feature_2_concept_id` | `integer \| null` | ✅ | Second feature concept identifier |
| `covariance_value` | `number` | ✅ | Covariance coefficient |

> **Note:** Pair uniqueness on `(feature_1_concept_id, feature_2_concept_id)` must be enforced at application level. Symmetric pairs are not deduplicated by the schema.

---

### 3.13 `Population`
Size and proportional descriptors for the cluster.

| Field | Type | Required | Constraints | Description |
|-------|------|----------|-------------|-------------|
| `n_count` | `integer` | ✅ | min: 10 | Absolute patient count |
| `profile_fraction` | `number` | ✅ | `[0, 1]` | Fraction of the parent profile population |
| `study_fraction` | `number` | ✅ | `[0, 1]` | Fraction of the total study population |

---

### 3.14 `QualityMetrics`
A collection of named validation metrics for the cluster.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `metrics` | `array<Metric>` (minItems: 1) | ✅ | List of metric entries |

#### 3.14.1 `Metric`

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `name` | `string` | ✅ | Metric name, e.g. `silhouette_score`, `True_Positives` |
| `value` | `number` | ✅ | Metric value |
| `metric_type` | `MetricType` | ✅ | Classification of the metric |

#### 3.14.2 `MetricType` Enum

| Value | Description |
|-------|-------------|
| `internal_cluster` | Internal cluster validity index, e.g. Silhouette score |
| `external_confusion` | Confusion matrix component, e.g. TP, FP, TN, FN |
| `external_mcc` | Matthews Correlation Coefficient |
| `class_distribution` | Patient count per class label within the cluster |

---

## 4. Standard Mappings

| Schema Element | External Standard |
|----------------|-------------------|
| `@context` | JSON-LD 1.1 |
| `global_metadata.author` | PAV `pav:authoredBy` |
| `provenance` | PAV `pav:createdOn`, `pav:lastUpdateOn`, `pav:version` |
| `dataset_metadata` | DCAT `Dataset` |
| `concept_id` / `unit_concept_id` | OMOP CDM / Athena (reference implementation; other vocabularies permitted) |
| `algorithmic_config` | MINIMAR reporting guideline |
