# DATA HANDLING AND INFRASTRUCTURE FOR AI

# Milestone 2 Report

## Project definition

### Project title
Dermalens Skin Cancer Classification Pipeline

### Problem statement
This project addresses a binary classification task in dermatology: predicting whether a skin lesion is malignant or benign using dermoscopic image features and patient metadata. The objective is to build a reproducible, auditable data-preparation and feature-engineering pipeline that can support later model training and evaluation.

### Intended users and setting
The project is designed for academic and research use by students, researchers, and practitioners working in applied machine learning for healthcare. It is intended as a transparent teaching and experimental workflow, not as a production clinical decision system. The setting is a research environment in which reproducibility, data governance, data quality checks, and valid feature preparation are essential before model development.

### Exact analytical task
The analytical task is a supervised classification problem:
- Input: dermoscopic lesion images and structured metadata such as age, sex, and anatomical location
- Target: binary label indicating malignant vs. benign lesion status
- Goal: produce cleaned, validated, and transformed feature tables suitable for future model training and evaluation

### Changes from Milestone 1
Milestone 1 established the problem framing, data source selection, storage design, and initial splitting strategy. Milestone 2 extends this by grounding the project in a documented end-to-end data pipeline, performing purposeful exploratory data analysis, validating the quality of the data, and preparing transformed feature tables. The central change from Milestone 1 is the shift from data acquisition and storage decisions to evidence-driven pipeline understanding and reproducible feature preparation.

### End-User Requirements
The primary users of this project are researchers, students, and practitioners working in an academic or training context on skin-lesion classification pipelines.

The project should allow a user to:
- reproduce the data-preparation workflow from source data to cleaned and validated feature tables;
- load the Dermalens dataset and check schema integrity, leakage, and data quality issues;
- generate processed outputs suitable for downstream model development;
- run the workflow in either Google Colab or a local Jupyter environment with clear instructions;
- understand the dataset balance, limitations, and engineering decisions before model training; and
- extend the workflow into a baseline model and evaluation pipeline in a later milestone.

This project is intended for transparent, reproducible research and teaching use. It is not presented as a clinical decision system and must be used with the documented governance and licence restrictions.

---

## Data source

### Source and provenance
- Dataset: Dermalens
- Platform: Hugging Face Datasets
- URL: https://huggingface.co/datasets/dheraingoud/dermalens-datasets
- Original source: HAM10000 dataset
- Pinned revision: 52bb1787931b301d610829724dc78c15c2cab797

### Repository evidence
- Project repository: https://github.com/tangerine3000/dh-infra-20122699-skin
- Project purpose, source, setup, and run instructions: [README.md](README.md)
- Data source and provenance notes: [docs/data_source.md](docs/data_source.md)
- Governance and licensing notes: [docs/governance_and_licensing.md](docs/governance_and_licensing.md)
- Reproducibility guide: [docs/reproducibility.md](docs/reproducibility.md)
- Main pipeline implementation notebook: [notebook-skin-2.ipynb](notebook-skin-2.ipynb)
- Repository configuration: [config.yaml](config.yaml)
- Dependency specification: [requirements.txt](requirements.txt)
- Project folder structure overview: [README.md](README.md)

### Data owner and licence
The underlying source data is derived from the HAM10000 public dermatology archive. The original HAM10000 dataset is licensed under Creative Commons Attribution-NonCommercial (CC-BY-NC 4.0). This project is therefore scoped to academic and educational use only, with no commercial deployment without separate rights and legal review.

### Collection context and permissions
The data was collected in a clinical dermatology context using dermoscopic imaging from multiple sources. The dataset contains lesion images and metadata, including diagnosis-related fields along with demographic variables such as patient age and sex. The dataset is publicly accessible through Hugging Face and is used here under the published source terms.

### Privacy, bias, and ethical considerations
The dataset is de-identified at source, but it still contains clinically relevant metadata and demographic information. This introduces potential bias related to under-representation of darker skin tones, class imbalance, and limited demographic variation in some subgroups. These risks matter because a model trained on biased distributions can perform poorly for under-represented groups. The project therefore treats the dataset as a research dataset rather than a clinical decision tool. Any future deployment would require human oversight, broader fairness validation, and regulatory review.

### Governance summary
The repository documents the dataset provenance, licence, use restrictions, and ethical limitations in the governance and licensing materials. The project explicitly avoids claiming medical readiness or autonomous clinical decision-making.

---

## Data description

### Unit of analysis
The unit of analysis is an individual skin lesion instance with associated metadata and image content. Each record represents a lesion image and associated patient/lesion metadata used to model malignancy risk.

### Number of records and fields
The project uses a skin-lesion dataset containing 13,354 images before deduplication. After data cleaning and deduplication, 10,015 rows were retained in the processed dataset used for Milestone 1 and carried forward into Milestone 2. The structured dataset includes fields such as:
- lesion image identifiers
- diagnosis label / target
- age
- sex
- lesion location
- source metadata and preprocessing identifiers

### Data types
The dataset contains both structured and image-based data:
- numeric: age, derived quality scores, and other tabular features
- categorical: sex, lesion location, source labels, and diagnosis-related metadata
- image: dermoscopic lesion images in standardized image formats

### Target variable
The main target variable is a binary malignancy label, where malignant classes are encoded as 1 and non-malignant classes as 0. This supports a binary classification task aligned with the project objective.

### Class balance and missingness
The data is imbalanced: approximately 19.5% malignant and 80.5% benign. Missingness is present in metadata fields, particularly age and sex, which must be handled explicitly. The source documentation also records missingness in lesion location and demographic metadata, making validation and imputation a key part of the pipeline.

### Known limitations
The main limitations are:
- class imbalance
- demographic imbalance and possible representation bias
- image quality and acquisition variation
- missing metadata values
- risk of leakage if multiple images from the same lesion are not tracked correctly

These issues directly influence the pipeline design, validation checks, and downstream feature engineering decisions.

---

## Exploratory data analysis

### EDA objectives
The Milestone 2 EDA is intended to answer practical questions that affect data quality and ML readiness:
- Is the target distribution stable and interpretable?
- Are there missing values or inconsistent metadata patterns?
- Are there duplicated or leaked records across partitions?
- Do the available features support a stable and fair model-learning process?

### Key EDA findings
The project documentation and notebook analysis identify the following issues and patterns:

1. Class imbalance
   - Malignant examples are a minority class.
   - This affects training behaviour, threshold selection, and evaluation strategy.
   - The imbalance drives the need for careful splitting and downstream metrics beyond simple accuracy.

2. Missingness in patient metadata
   - Age is missing in a non-trivial proportion of samples.
   - Sex is also incomplete, and other metadata can be sparse.
   - Missingness is not neutral: it affects feature engineering and may bias population summaries if mishandled.

3. Duplicates and leakage risk
   - The project explicitly removes exact duplicates and preserves lesion integrity during splitting.
   - This is critical because multiple images from the same lesion can otherwise leak information between train and test partitions.

4. Distributional patterns by demographic and lesion metadata
   - The EDA examines distributions by age, lesion site, and class label to understand possible imbalance and representation issues.
   - These checks are necessary because the dataset is not a clean, perfectly balanced academic toy dataset; it reflects real clinical variation.

### Interpretation and impact on pipeline decisions
The EDA is not a checklist of summary statistics alone; it directly informs the engineering choices:
- missing age values require an explicit imputation strategy
- sex and lesion-site metadata require encoding decisions
- stratification and lesion-aware splitting are necessary to minimise leakage
- class imbalance requires continued attention during evaluation and model selection

These decisions make the EDA a central driver of the pipeline rather than a decorative appendix.

### Visualisation and interpretation
The repository includes EDA outputs and code-based analysis in [notebook-skin-2.ipynb](notebook-skin-2.ipynb). These outputs are interpreted as evidence of the data quality issues that drive the pipeline decisions.

### Repository evidence for required items
- README with project purpose, data source, environment setup, and exact run instructions: [README.md](README.md)
- Dependency specification: [requirements.txt](requirements.txt)
- Data acquisition and validation workflow: [notebook-skin.ipynb](notebook-skin.ipynb) and [notebook-skin-2.ipynb](notebook-skin-2.ipynb)
- Data cleaning and transformation stages: [notebook-skin-2.ipynb](notebook-skin-2.ipynb)
- EDA: [notebook-skin-2.ipynb](notebook-skin-2.ipynb)
- Clear folder structure and project layout: [README.md](README.md)
- Repository history and commit trail: GitHub repository history at https://github.com/tangerine3000/dh-infra-20122699-skin
 includes EDA outputs and notebook analysis covering distributions, class balance, and metadata quality. These visualisations are interpreted in the notebook and used to justify data-cleaning and feature-engineering choices. The analytic emphasis is on whether the data supports a stable, fundamental preprocessing pipeline before model training begins.---

## Design / architecture

### Data-flow diagram
```text
Source (Dermalens / HAM10000-derived dataset)
    ↓
Ingest
  - load Hugging Face dataset and config.yaml
  - read parquet and metadata files into the notebook pipeline
    ↓
Validate
  - schema and type checks
  - required-field validation
  - duplicate removal and leakage checks
  - range and missingness validation
    ↓
Store
  - Google Cloud Storage (GCS) prefixes such as datasets/v1/
  - processed parquet outputs in versioned project folders
    ↓
Transform
  - EDA-driven cleaning and feature engineering
  - age imputation, scaling, label derivation, and encoding
  - image metadata preprocessing
    ↓
Output
  - train/dev/test feature tables
  - metadata files and summary artifacts
  - data ready for Milestone 3 model training
```

This diagram matches the implemented project workflow and storage choices used in the repository: the pipeline begins from the Dermalens source, ingests the dataset through the notebook, validates and cleans the data, stores outputs in versioned GCS/local parquet structures, transforms features for modelling, and generates reproducible output artifacts. It is therefore linked to the actual project implementation rather than a generic cloud architecture.

### Automation and manual steps
The project is semi-automated in a realistic academic workflow:
- Automated:
  - data loading
  - validation checks
  - duplicate cleanup and partition logic
  - feature engineering steps
  - storage of processed outputs
- Manual / human-controlled:
  - data-source selection and governance review
  - interpretation of EDA findings
  - confirmation of dataset revision and project configuration
  - verification of integrity before model development

This architecture reflects the actual implementation in the repository and matches the storage and processing decisions recorded in the project documentation.

---

## Implementation

### Evidence that the pipeline runs
The repository includes a working notebook-based workflow for data acquisition, validation, cleaning, and transformation. The implementation covers the M1-to-M2 pipeline progression and shows the data path from raw source to processed parquet outputs.

The project stores data using versioned structures in Google Cloud Storage and local processed directories. The source documentation records the data layout and major storage prefixes used in the project, including:
- raw dataset footprint
- combined data outputs
- sharded train/dev/test partitions
- manifest metadata

### Representative validation evidence
The implementation includes checks such as:
- schema validation for required fields
- consistency checks for metadata quality
- duplicate removal
- lesion-aware split validation to avoid leakage
- class-balance checks across train/dev/test partitions

### Reproducibility evidence
The project includes:
- `requirements.txt` for environment dependencies
- `config.yaml` for configuration settings
- reproducibility notes in the repository documentation
- pinned dataset revision information
- a notebook-based execution path for local or Colab use

### Practical output examples
The expected output includes processed artifacts such as:
- `data/processed/m2_outputs/features_train.parquet`
- `data/processed/m2_outputs/features_dev.parquet`
- `data/processed/m2_outputs/features_test.parquet`
- `data/processed/m2_outputs/metadata.json`

These artifacts show the pipeline is not only designed but operationally integrated with the repository workflow.

---

## Next steps

The main unresolved issues are practical rather than conceptual:
- continue to monitor and document class imbalance and fairness across demographic groups
- refine missing-data strategy where feature quality requires more explicit handling
- validate the final processed outputs on a fresh environment if the project is prepared for submission
- develop a baseline model in Milestone 3 using the cleaned feature tables and evaluation procedure


---

## Summary
This Milestone 2 report demonstrates that the project is well-structured around a clear classification task, a justified data source, documented governance, and a reproducible pipeline. The EDA is used to influence real engineering decisions rather than simply to display summary statistics. The repository therefore provides the evidence required for a robust academic data-handling and infrastructure project in the context of skin-cancer classification.

---

## AI Use Acknowledgement

This project was developed with assistance from AI tools to support notebook structure, code drafting, documentation, and troubleshooting. Project design prompt history used during development is recorded in the [Prompts folder](docs/prompts/).