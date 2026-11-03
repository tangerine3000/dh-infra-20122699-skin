# Dermalens Skin Cancer Classification Pipeline

**Project:** Data Handling & Infrastructure for AI  
**Task:** Predict skin lesion malignancy from dermoscopic images and patient metadata  
**Status:** Milestone 2 (Data Understanding & Feature Engineering)

## Quick Links

**Repository**
- [GitHub Repository](https://github.com/tangerine3000/dh-infra-20122699-skin)

**Milestone 1**
- [View M1 Notebook](notebook-skin.ipynb)
- [Open M1 in Colab](https://colab.research.google.com/github/tangerine3000/dh-infra-20122699-skin/blob/main/notebook-skin.ipynb)

**Milestone 2**
- [View M2 Notebook](notebook-skin-2.ipynb)
- [Open M2 in Colab](https://colab.research.google.com/github/tangerine3000/dh-infra-20122699-skin/blob/main/notebook-skin-2.ipynb)
- [Milestone 2 peer grading rubric](Milestone-2-Peer-Grading-Rubric-Dermalens.md)
- [Project definition and requirements](#end-user-requirements)
- [Data source and provenance](docs/data_source.md)
- [Governance and licensing](docs/governance_and_licensing.md)
- [Pipeline architecture and storage design](docs/pipeline_architecture.md)
- [Reproducibility guide](docs/reproducibility.md)
- [Dataset README](Dataset_README.md)

---

## AI Use Acknowledgement

This project was developed with assistance from AI tools to support notebook structure, code drafting, documentation, and troubleshooting. Prompt history used during development is recorded in the [Prompts folder](docs/prompts/).

## Purpose

This project builds an end-to-end machine-learning pipeline that predicts skin lesion malignancy from:
- dermoscopic lesion images;
- age, sex, and anatomical location metadata; and
- a binary target label for malignant vs. benign classification.

The workflow is designed around reproducibility, cloud storage, and transparent data governance.

## Project Overview

This repository contains a notebook-based workflow for:
- loading and validating the Dermalens dataset;
- deriving a binary target label (`label = 1` for malignant classes, `0` otherwise);
- checking for duplicates and leakage across train/dev/test partitions;
- performing exploratory data analysis and feature engineering;
- preparing reproducible feature tables for downstream modeling; and
- storing outputs in a versioned, auditable structure.

## End-User Requirements

The primary users of this project are researchers, students, and practitioners working in an academic or training context on skin-lesion classification pipelines.

The project should allow a user to:
- reproduce the data-preparation workflow from source data to cleaned and validated feature tables;
- load the Dermalens dataset and check schema integrity, leakage, and data quality issues;
- generate processed outputs suitable for downstream model development;
- run the workflow in either Google Colab or a local Jupyter environment with clear instructions;
- understand the dataset balance, limitations, and engineering decisions before model training; and
- extend the workflow into a baseline model and evaluation pipeline in a later milestone.

This project is intended for transparent, reproducible research and teaching use. It is not presented as a clinical decision system and must be used with the documented governance and licence restrictions.

## Governance and Licensing Summary

The source data was derived from a public medical-imaging dataset and is subject to the dataset licence and terms of use. The repository documentation records the source provenance, access route, and restrictions in [docs/governance_and_licensing.md](docs/governance_and_licensing.md).

Key points:
- Source dataset: Dermalens / HAM10000-derived public skin lesion data
- Primary licence: CC-BY-NC 4.0 at source level
- Intended use: educational, academic, and research workflows
- Non-commercial use restriction: this project should not be used for commercial deployment without separate rights and legal review
- Clinical status: research-only; not validated for autonomous medical decision-making

## Architecture and Data Flow

This project implements the following end-to-end workflow, matching the actual code and storage choices in the repository:

Source → Ingest → Validate → Store → Transform → Output

```text
Source (Dermalens / HAM10000-derived data)
    ↓
Ingest
  - Download/load parquet and metadata files
  - Load config.yaml and project settings
    ↓
Validate
  - schema checks
  - required-field and type validation
  - duplicate and leakage checks
  - audit of missingness and range constraints
    ↓
Store
  - GCS object storage and versioned data prefixes
  - local/processed parquet outputs for reproducible runs
    ↓
Transform
  - EDA and feature-quality analysis
  - image preprocessing and metadata engineering
  - tabular feature encoding, scaling, and imputation
    ↓
Output
  - train/dev/test feature tables
  - metadata and EDA summary artifacts
  - ready for downstream modelling in Milestone 3
```

For a fuller project-level design description and pipeline overview, see [docs/pipeline_architecture.md](docs/pipeline_architecture.md).

## Reproducibility and Setup

### Prerequisites

1. Python 3.10 or 3.11
2. Jupyter Notebook or JupyterLab
3. A Google Cloud project and/or local data directory with the expected M1 outputs

### Install dependencies

```bash
python -m venv .venv
source .venv/bin/activate  # macOS/Linux
.venv\Scripts\activate     # Windows
pip install -r requirements.txt
```

### Run the notebooks

**Option A: Google Colab**

1. Open the notebook in Colab with the GitHub URLs above.
2. Authenticate to Google Cloud if accessing GCS data:

```python
from google.colab import auth
auth.authenticate_user()
```

3. Run all cells from top to bottom.

**Option B: Local Jupyter**

```bash
jupyter notebook notebook-skin-2.ipynb
```

Or:

```bash
jupyter lab notebook-skin-2.ipynb
```

## Milestone 2 Outputs

After running the notebook, expected output artifacts include:
- `data/processed/m2_outputs/features_train.parquet`
- `data/processed/m2_outputs/features_dev.parquet`
- `data/processed/m2_outputs/features_test.parquet`
- `data/processed/m2_outputs/metadata.json`
- EDA visualization files such as `eda_class_distribution.png`

## Project Structure

```text
.
├── README.md
├── requirements.txt
├── config.yaml
├── notebook-skin.ipynb
├── notebook-skin-2.ipynb
├── Dataset_README.md
├── docs/
│   ├── governance_and_licensing.md
│   ├── reproducibility.md
│   ├── prompts/
│   └── ...
├── images/
└── data/
```

## Additional Documentation

This repository includes the main evidence required by the Milestone 2 rubric:

- [Project definition and end-user requirements](#end-user-requirements)
- [Data source and provenance](docs/data_source.md)
- [Governance and licensing](docs/governance_and_licensing.md)
- [Pipeline architecture and storage design](docs/pipeline_architecture.md)
- [Exploratory data analysis and implementation notebook](notebook-skin-2.ipynb)
- [Reproducibility guide](docs/reproducibility.md)
- [Milestone 2 peer grading rubric](Milestone-2-Peer-Grading-Rubric-Dermalens.md)
- [Dataset README](Dataset_README.md)
- [Prompt archive](docs/prompts/)

This repository is intended to be a transparent, auditable, and reproducible research pipeline rather than a production clinical system.
