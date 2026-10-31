# Project Definition: Skin Lesion Malignancy Classification

## Overview

**Project Title:** Dermalens Skin Cancer Detection Pipeline

**Problem Statement:** Develop a machine learning system that can predict the malignancy risk of skin lesions from dermoscopic images combined with patient demographic and clinical features.

## Task Definition

**Analytical Task:** Binary image classification with tabular feature fusion

**Input:**
- Dermoscopic lesion images (RGB, variable resolution)
- Patient age (years)
- Patient sex (male, female, unknown)
- Anatomical location of lesion (15 body sites)

**Output:**
- Binary classification: 1 = malignant (requires intervention), 0 = benign

**Malignant Classes:**
- Melanoma
- Basal cell carcinoma
- Actinic keratoses

**Benign Classes:**
- Melanocytic nevi
- Benign keratosis-like lesions
- Dermatofibroma
- Vascular lesions

## Intended Users & Setting

**Primary Users:**
- Dermatologists (preliminary screening aid, not replacement for clinical judgment)
- Telemedicine platforms (triage for resource allocation)
- Public health programs (resource-constrained settings)

**Use Case:**
- Assist with early detection of potentially malignant lesions
- Reduce unnecessary biopsies by identifying benign cases with high confidence
- Support clinical decision-making at point of care

**Success Criteria:**
- AUC-ROC ≥ 0.85 on held-out test set
- High sensitivity (minimize false negatives for malignant cases)
- Interpretable predictions with feature importance

## Data Source & Collection

**Dataset:** Dermalens (Hugging Face) - derived from HAM10000 database

**Collection Context:**
- Dermoscopic imaging study
- Diverse patient demographics and lesion presentations
- Class imbalance: ~19.5% malignant, ~80.5% benign (before deduplication)

## Changes from Milestone 1

**M1 Deliverables:**
- Confirmed binary classification task
- Data source and licence verified
- Reproducible pipeline for data acquisition and splitting
- Train/dev/test splits (80/10/10) with lesion-based grouping to prevent leakage

**M2 Additions (This Milestone):**
- Comprehensive EDA of image properties and tabular features
- Image preprocessing pipeline (standardization, validation)
- Tabular feature engineering (scaling, encoding)
- Data quality assessment and validation
- Documentation of design decisions
- Prepared feature matrices ready for model training

**M3 Planned:**
- Baseline model training (CNN + tabular fusion)
- Model evaluation and interpretation
- Refinement and optimization

## Key Principles

1. **No Leakage:** Lesion IDs are grouped across splits; validation/test data never influences training
2. **Interpretability:** Every preprocessing step is justified and documented
3. **Reproducibility:** All code runs without manual intervention from public sources
4. **Data-Driven:** EDA findings directly inform design decisions
