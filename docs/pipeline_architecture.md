# Pipeline Architecture & Design Decisions

## Overall Data Pipeline

### Milestone 1 → Milestone 2 → Milestone 3

```
┌─────────────────────────────────────────────────────────────────┐
│                      MILESTONE 1                                 │
│              Data Acquisition & Splitting                        │
├─────────────────────────────────────────────────────────────────┤
│ Hugging Face                                                     │
│ ↓                                                               │
│ Download raw parquet (train/val/test)                          │
│ ↓                                                               │
│ Combine splits + deduplicate + derive label                    │
│ ↓                                                               │
│ GCS: raw/ (original source)                                    │
│ GCS: combined_data/ (deduplicated, labelled)                   │
│ ↓                                                               │
│ Lesion-grouped stratified split → train/dev/test (80/10/10)    │
│ 8-fold stratified CV from training set                         │
│ ↓                                                               │
│ GCS: sharded/ (train.parquet, dev.parquet, test.parquet)       │
│ GCS: manifest.json (metadata, versions)                        │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      MILESTONE 2                                 │
│        Data Understanding & Feature Engineering                 │
├─────────────────────────────────────────────────────────────────┤
│ Input: GCS sharded/ (train/dev/test parquet)                    │
│ ↓                                                               │
│ CELL 1-2: Setup & Load                                         │
│  - Install dependencies                                        │
│  - Load config.yaml                                            │
│  - Load train/dev/test from GCS or local                       │
│ ↓                                                               │
│ CELL 3: Data Profile                                           │
│  - Basic statistics                                            │
│  - Missing values per column                                   │
│  - Class distribution                                          │
│ ↓                                                               │
│ CELL 4: Validation                                             │
│  - Schema checks (required fields, types)                      │
│  - Constraint checks (ranges, valid values)                    │
│  - Duplicates, missing patterns                                │
│  - No lesion leakage across partitions                         │
│  - Output: validation report                                   │
│ ↓                                                               │
│ CELL 5: Exploratory Data Analysis (EDA)                        │
│  - Image properties (sample images, dimensions, pixel stats)   │
│  - Tabular feature distributions (age, sex, location)          │
│  - Class balance analysis (overall + by sex/location)          │
│  - Correlations & missing patterns                             │
│  - Interpretation: how findings inform decisions               │
│ ↓                                                               │
│ CELL 6: Image Preprocessing                                    │
│  - Load images from parquet                                    │
│  - Validate (format, size, readability)                        │
│  - Standardize dimensions (resize/pad to 224×224)              │
│  - Normalize pixel values (0-1)                                │
│  - Log any failures                                            │
│ ↓                                                               │
│ CELL 7: Tabular Feature Engineering                            │
│  - Scale age (z-score)                                         │
│  - Encode sex (one-hot: male, female, unknown → 3 features)    │
│  - Encode location (one-hot: 15 anatomical sites)              │
│  - Impute missing (documented strategy)                        │
│ ↓                                                               │
│ CELL 8: Feature Validation                                     │
│  - Check no nulls in final features                            │
│  - Verify distributions per split                              │
│  - Confirm class balance maintained                            │
│ ↓                                                               │
│ CELL 9: Data Splitting (Verification)                          │
│  - Confirm M1 split integrity                                  │
│  - Check lesion_id no leakage                                  │
│  - Verify stratification on label                              │
│ ↓                                                               │
│ CELL 10: Output & Summary                                      │
│  - Save feature matrices (train/dev/test) to parquet           │
│  - Save image metadata (dimensions, stats)                     │
│  - Write manifest (preprocessing params, versions)             │
│  - Summary: ready for model training                           │
│ ↓                                                               │
│ Output: data/processed/m2_outputs/                             │
│  - features_train.parquet                                      │
│  - features_dev.parquet                                        │
│  - features_test.parquet                                       │
│  - image_metadata.parquet                                      │
│  - metadata.json                                               │
└─────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────┐
│                      MILESTONE 3                                 │
│         Baseline Model Training & Evaluation                     │
├─────────────────────────────────────────────────────────────────┤
│ Input: M2 feature matrices + preprocessed images                │
│ ↓                                                               │
│ Build CNN + tabular fusion model                               │
│ Train on M2 train split                                        │
│ Evaluate on dev/test                                           │
│ Compute AUC-ROC, precision, recall, F1                         │
└─────────────────────────────────────────────────────────────────┘
```

## Storage Design Decisions

### Why Google Cloud Storage (GCS)?

**Choice:** Parquet files in GCS object store (not a database)

**Justification:**

1. **Access Pattern:**
   - Write-once, read-whole-file (never queried row-by-row)
   - Metadata is small (fits in memory)
   - No concurrent writes needed

2. **Cost & Scalability:**
   - Object store is cheaper than database per GB
   - No server management
   - Direct integration with Colab/Vertex AI

3. **Format (Parquet):**
   - Columnar: metadata read without image bytes
   - Typed schema survives round trips (unlike CSV)
   - Compressed (smaller disk footprint)
   - Supports nested structures (images as bytes)

4. **Versioning:**
   - Each version under `datasets/<version>/`
   - Older versions never overwritten
   - Manifest tracks all parameters

**Alternatives Considered:**
- BigQuery: Overkill (no SQL queries needed); higher cost
- PostgreSQL: Unnecessary (no relational structure); operational overhead
- Local filesystem: Works for M2; doesn't scale for M3 (training jobs can't access local files)

**Final Choice:** GCS Parquet (proven in M1; cost-effective; scales to model training)

## Data Splitting Strategy

### Why Lesion-Grouped Stratified Split?

**Problem:** HAM10000 has multiple images per lesion. Random split would create leakage (same lesion in train & test).

**Solution:** StratifiedGroupKFold with:
- **Stratification Target:** `label` (binary, 0/1)
- **Grouping Key:** `lesion_id` (all images of same lesion stay together)

**Result:**
- Train / Dev / Test = 80% / 10% / 10%
- Class balance preserved (malignant rate ~19.5% in each)
- No lesion overlap across partitions
- 8-fold cross-validation from training set only

## Image Preprocessing Pipeline

### Design: Standardize Without Loss

**Steps:**

1. **Load:** Images stored as bytes in parquet → PIL Image
2. **Validate:** Format (JPG/PNG), size, readability
3. **Resize:** 224×224 (standard for pre-trained models)
   - Method: center-preserve aspect ratio with padding
   - Rationale: maintains clinical features; compatible with MobileNetV2
4. **Normalize:** Pixel values 0-255 → 0.0-1.0
   - Rationale: standard input for neural networks
5. **No Augmentation:** M2 only validates pipeline; augmentation in M3

**Why Not Part of Parquet?**
- Image preprocessing is model-dependent
- Resizing ↔ normalization parameters should be stored with model, not data
- M2 validates preprocessing works; M3 applies it during training

## Tabular Feature Engineering

### Features

| Feature | Type | Source | Transformation | Reason |
|---------|------|--------|-----------------|--------|
| age | numeric | source `age` | z-score normalize, median impute | Clinical relevance; malignancy risk increases with age |
| sex | categorical | source `sex` | one-hot encode, handle missing | Some lesion types more common in one sex |
| localization | categorical | source `localization` | one-hot encode, handle missing | Location strongly correlates with lesion type (e.g., face → actinic keratoses) |

### Rationale for Tabular Features

- **Images alone are strong** but clinical features (age, sex, location) improve accuracy
- **Fusion approach:** CNN extracts image features; dense layer processes tabular features; concatenate for final prediction
- **Interpretability:** Tabular features are directly readable; helps clinicians understand decisions

### Why One-Hot Encoding?

- Sex: only 2-3 categories (male, female, unknown) → one-hot is clear
- Location: 15 anatomical sites → one-hot preserves all information without imposing order
- Alternative (ordinal): No meaningful order to locations

## Validation Strategy

### What & Why

**Schema Validation:**
- Required fields present
- Correct data types
- No unexpected columns

**Constraint Validation:**
- Age: 0-150 years (biological range)
- Label: only 0 or 1
- Partition: only train/dev/test

**Leakage Detection:**
- No `lesion_id` overlap across train/dev/test
- No `lesion_id` on both sides of CV fold

**Consistency:**
- Class balance within 5% across partitions
- No rows lost in preprocessing

### Automated vs. Manual

**Automated (M2 notebook):**
- Schema & constraint checks (assertions)
- Leakage detection
- Statistics logging

**Manual (EDA + interpretation):**
- Visual inspection of sample images
- Distribution analysis and decision-making
- Design justification

---

## Summary: Design Principles

1. **Lesion Integrity:** All images of same lesion stay together
2. **Stratification:** Class balance preserved across splits
3. **No Leakage:** Validation/test never seen during preprocessing fitting
4. **Reproducible:** Seed=42, pinned data versions, documented params
5. **Interpretable:** Every preprocessing step justified in EDA
6. **Scalable:** GCS storage integrates with model training infrastructure
