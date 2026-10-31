# Feature Engineering & Tabular Features

## Overview

Image features come directly from dermoscopic images (preprocessed in M2, engineered by CNN in M3). This document covers **tabular features** derived from structured patient and lesion metadata.

## Feature Specifications

### Age Feature

**Source:** Metadata field `age` (patient age in years)

**Transformation:**
1. **Imputation:** Median imputation (fill missing with training set median)
   - Missing rate: ~8%
   - Justification: median is robust to outliers; preserves distribution
2. **Scaling:** Z-score normalization
   - Formula: `(age - train_mean) / train_std`
   - Why: Neural networks learn better with zero-mean, unit-variance inputs
3. **Validation:** Range 0-150 years (check for data entry errors)

**Clinical Relevance:**
- Melanoma risk increases with age
- Benign nevi common in younger patients
- Age × location interaction possible (e.g., actinic keratoses on face in older patients)

**Example:**
- Age 45: z-score = (45 - train_mean) / train_std
- If train_mean = 50, train_std = 20: z-score = (45-50)/20 = -0.25

### Sex Feature

**Source:** Metadata field `sex` (patient sex)

**Categories:**
- `male`: 1 if male, 0 otherwise
- `female`: 1 if female, 0 otherwise
- `unknown`: 1 if missing/unknown, 0 otherwise

**Transformation:**
1. **Handle Missing:** Replace with `unknown`
   - Missing rate: ~2%
2. **Encode:** One-hot encoding (3 binary columns)
   - Why: No ordinal relationship; one-hot is standard for categorical features
   - Output: 3 columns (overcomplete, but interpretable)

**Clinical Relevance:**
- Some lesion types more common in one sex
- Basal cell carcinoma slightly more common in males
- Melanoma shows complex sex/age interactions

**Example:**
```
Input:  sex = "male"
Output: [male=1, female=0, unknown=0]

Input:  sex = NaN
Output: [male=0, female=0, unknown=1]
```

### Localization Feature

**Source:** Metadata field `localization` (body site of lesion)

**Anatomical Sites (15 categories):**
- back
- chest
- ear
- face
- forearm
- hands
- legs
- lower extremity
- neck
- scalp
- torso
- upper extremity
- foot
- lower back
- unknown/missing

**Transformation:**
1. **Handle Missing:** Replace with `unknown`
   - Missing rate: <1%
2. **Encode:** One-hot encoding (15 binary columns)
   - Why: Location strongly predicts lesion type; no ordinal relationship

**Clinical Relevance:**
- Actinic keratoses concentrated on face, scalp, ears (sun-exposed)
- Melanoma more common on back (males), legs (females)
- Different risk profiles by location; guides dermatologist attention

**Example:**
```
Input:  localization = "face"
Output: [back=0, chest=0, ..., face=1, ..., unknown=0]

Input:  localization = NaN
Output: [back=0, chest=0, ..., face=0, ..., unknown=1]
```

## Feature Engineering Decisions

### Why These Three Features?

1. **Age:** Directly impacts melanoma risk; standard clinical feature
2. **Sex:** Modulates risk for some lesion types; easily available
3. **Location:** Strong predictor of lesion type and risk; no extra cost to collect

### Why Not Other Features?

- `dx_type` (how diagnosis confirmed): Would leak information unavailable for new patients
- `image_id`, `lesion_id`: Identifiers only; no predictive power
- `dx` (original diagnosis): This is the source of the target; would be leakage

### Fusion with Image Features

**M3 Architecture (CNN + Tabular):**

```
Images → CNN → Image Embeddings (e.g., 2048-dim)
                    ↓
                  Concatenate
                    ↓
Tabular Features → Dense Layer → Tabular Embeddings (e.g., 64-dim)
                    ↓
              Final Dense Layer (2 hidden units)
                    ↓
              Sigmoid → Probability [0,1]
```

**Rationale:**
- Image features dominant (rich information from visual data)
- Tabular features provide clinical context
- Fusion improves accuracy by ~3-5% in similar tasks

## Validation & Quality Checks

### During Feature Engineering (M2 Cell 7)

1. **No Missing Values After Imputation:**
   - Age: imputed with median
   - Sex & location: missing → `unknown`
   - Assert all non-null after transformation

2. **Feature Ranges:**
   - Age z-score: typically -3 to +3 (check outliers)
   - One-hot: all binary (0 or 1)
   - Exactly one category per record (e.g., sex columns sum to 1)

3. **Consistency Across Splits:**
   - Imputation params (median age) fit on training only
   - Applied identically to dev/test
   - Dev/test must not influence imputation

### Example Validation Code

```python
# Age: check for missing after imputation
assert train['age'].notna().all()
assert dev['age'].notna().all()
assert test['age'].notna().all()

# Sex: check one-hot is binary
sex_cols = [col for col in features.columns if col.startswith('sex_')]
assert (features[sex_cols].isin([0, 1])).all().all()

# Location: check all records have exactly one location
assert features[[col for col in features.columns if col.startswith('location_')]].sum(axis=1).eq(1).all()
```

## Summary

**Total Tabular Features:**
- Age: 1 numeric (z-scored)
- Sex: 3 binary (one-hot: male, female, unknown)
- Location: 15 binary (one-hot: 15 sites)
- **Total: 19 tabular features**

These combine with image features (thousands of dimensions from CNN) in the final model to predict malignancy.
