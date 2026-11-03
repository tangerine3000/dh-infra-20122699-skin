# Governance, Licensing, and Ethical Use

## 1. Data source and provenance

This project uses the Dermalens dataset hosted on Hugging Face, derived from the HAM10000 skin lesion archive. The repository documentation identifies the source as the public Dermalens dataset and records the pinned Hugging Face revision used for reproducibility.

Key provenance details:
- Source platform: Hugging Face Datasets
- Dataset: `dheraingoud/dermalens-datasets`
- Original dataset family: HAM10000
- Reproducibility note: use the dataset revision recorded in the project documentation and in the notebook metadata

## 2. Licensing and permitted use

The underlying source dataset is governed by a Creative Commons Attribution-NonCommercial (CC-BY-NC 4.0) licence. This means:
- Academic and educational use is permitted.
- Commercial use is not permitted without additional rights.
- Attribution to the original dataset and source publication is required.

This repository is intended for educational, academic, and research usage in line with those restrictions. It should not be used for commercial deployment or direct clinical decision support without separate legal review and appropriate licensing.

## 3. Privacy and ethics

The dataset contains de-identified dermoscopic images and metadata including age, sex, and lesion location. While names and direct identifiers are not exposed in the curated dataset, clinical and demographic risk remains:
- the dataset reflects the distribution of the source clinical population, which may underrepresent some groups;
- demographic imbalance can affect fairness and generalisation; and
- model outputs are not a substitute for professional medical judgment.

The project should therefore be treated as a research workflow rather than a clinical decision system. Any real-world deployment should require additional validation, clinician oversight, and bias review.

## 4. Storage and access governance

The project stores derived data in Google Cloud Storage (GCS) under a versioned prefix and keeps the raw source files separate from processed files. This reduces accidental overwrite and makes data provenance easier to audit.

The design follows a practical governance posture:
- raw data is kept unchanged;
- processed data is versioned by prefix (for example `datasets/v1/`);
- output artifacts are documented and reproducible; and
- no secrets or credentials are stored in the repository.

## 5. Reproducibility requirements

To maintain reproducibility and reduce operational risk:
- keep the pinned dataset revision documented;
- keep environment dependencies pinned in `requirements.txt`;
- keep `config.yaml` aligned with the intended local or GCS data path; and
- avoid overwriting prior dataset versions.

## 6. Recommended project policy

For future work, the project should keep a short governance note in every milestone release:
- source of data;
- licence and permitted use;
- risk and limitations;
- intended scope of the model; and
- whether the work is academic-only or open for broader use.

This project is best framed as an educational and research pipeline for non-commercial skin-lesion analysis. 
