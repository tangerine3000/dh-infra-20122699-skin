# M2 Implementation Prompts

Record of AI prompts used to generate Milestone 2 deliverables.

---

## Initial Planning

**Prompt 1: Caveman Plan**
```
Without modifying existing files, plan code to meet the key focus areas 
and to include supplemental files to explain
```
→ Generated: Initial plan for notebook + design docs

**Prompt 2: Notebook-Based Approach**
```
can this be done in a jupyter notebook?
```
→ Confirmed: Single notebook approach with supporting docs

**Prompt 3: Complete Plan**
```
plan to create solution in a notebook called notebook-skin-2
```
→ Generated: Detailed 11-cell notebook structure

**Prompt 4: Spec Coverage Check**
```
does this plan meet all the spec requirements
```
→ Identified gaps: storage design, baseline model, next steps docs

**Prompt 5: Updated Complete Plan**
```
using the plan, implement
```
→ Triggered: Full implementation of all files

---

## File-Specific Prompts

### config.yaml
**Implicit:** Configuration values from M2 spec
- Image preprocessing params (224×224, normalize)
- Feature engineering (age z-score, categorical encoding)
- Validation rules, output paths
- Baseline model placeholders

### docs/project_definition.md
**Implicit:** From M2 spec section 1.1 + user selection
- Project title: Dermalens Skin Cancer Classification
- Task: Binary image classification with tabular features
- Input: dermoscopic images + age, sex, location
- Output: malignant/benign prediction
- Users: Dermatologists, telemedicine, public health
- Success criteria: AUC ≥ 0.85

### docs/data_source.md
**Implicit:** From M2 spec section 1.1
- Hugging Face Dermalens dataset
- HAM10000 original source
- CC-BY-NC 4.0 licence (non-commercial)
- Privacy: anonymized, no PII beyond age/sex/location
- Ethical: demographic bias, clinical validation needed
- Data quality: ~19.5% malignant, ~80.5% benign

### docs/pipeline_architecture.md
**Implicit:** From M2 spec "Design/Architecture" + implementation
- Diagram: Source → M1 (split) → M2 (EDA, features) → M3 (train)
- Storage design: GCS + Parquet (justification vs. DB/local)
- Splitting strategy: Lesion-grouped, stratified on label
- Image preprocessing: resize 224×224, normalize 0-1
- No augmentation in M2

### docs/feature_engineering.md
**Implicit:** From feature specs
- Age: z-score normalize, median impute
- Sex: one-hot encode (male, female, unknown)
- Location: one-hot encode (15 anatomical sites)
- Rationale: clinical relevance, model input standards
- Validation: check ranges, no nulls, consistency

### docs/storage_design.md
**Implicit:** Cloud storage evaluation
- Decision: GCS + Parquet (chosen)
- Why: Cost, scalability, integration with Colab/Vertex
- Alternatives: BigQuery (overkill), PostgreSQL (overhead), local (doesn't scale)
- Cost estimate: <$1/month for this project
- Manifest approach: versioning & reproducibility

### docs/baseline_model.md
**Implicit:** From M3 planning section of spec
- Architecture: MobileNetV2 (image) + tabular fusion
- Loss: BCE with class weights {0: 0.8, 1: 4.1}
- Metrics: AUC, precision, recall, F1
- Hyperparameters: LR 1e-3, batch 32, dropout [0.5, 0.3]
- Training: 50 epochs, early stopping on dev AUC
- Target: AUC ≥ 0.85, Recall ≥ 0.85

### docs/next_steps.md
**Implicit:** From M2 spec "Next Steps (Optional)" + limitations
- Unresolved: class imbalance, image quality, missing values
- Baseline plan: CNN + tabular, weighted loss, augmentation
- M3 evaluation: cross-validation, per-subgroup analysis
- Success criteria: AUC ≥ 0.85, recall ≥ 0.85
- Deliverables: model, training history, metrics, report

### notebook-skin-2.ipynb
**Explicit Structure:**
```
Cell 1:  Setup & Imports
         - Libraries, config load, GCS/local paths

Cell 2:  Data Acquisition
         - Load M1 train/dev/test from GCS or local
         - Verify all splits present

Cell 3:  Data Profile
         - Basic stats, missingness, class distribution
         - Per-split analysis

Cell 4:  Validation
         - Schema checks (required fields, types)
         - Constraint validation (ranges, valid values)
         - Leakage detection (lesion_id overlap)

Cell 5:  EDA (CENTRAL - interpreted)
         - Class distribution + visualization
         - Age analysis + by-class breakdown
         - Sex analysis + class breakdown
         - Location analysis + malignancy rates
         - Image sampling (benign vs malignant)
         - **Every finding interpreted & linked to decisions**

Cell 6:  Image Preprocessing
         - Load image from parquet
         - Validate (format, size, readability)
         - Resize to 224×224 (center-preserve aspect ratio)
         - Normalize to [0, 1]
         - Log failures

Cell 7:  Tabular Feature Engineering
         - Age: median impute, z-score normalize
         - Sex: fill missing → 'unknown', one-hot encode
         - Location: fill missing → 'unknown', one-hot encode
         - Combine into feature matrix

Cell 8:  Feature Validation
         - Check no nulls after engineering
         - Verify one-hot binary (0/1)
         - Check consistency across splits
         - Assert all checks pass

Cell 9:  Data Splitting Verification
         - Confirm M1 split integrity
         - Check lesion_id no leakage
         - Verify stratification on label
         - Report split sizes & class balance

Cell 10: Output & Summary
         - Save features to parquet (train/dev/test)
         - Save metadata.json (preprocessing params)
         - Print completion summary
         - Ready for M3

Cell 11: Next Steps
         - Unresolved data issues
         - Baseline model plan
         - M3 success criteria
         - Documentation references
```

### requirements.txt
**Implicit:** From dependencies + M1
- Existing: pandas, numpy, pyarrow, scikit-learn, google-cloud-storage, jupyter, datasets, Pillow, matplotlib
- Added for M2:
  - google-cloud-auth (GCS auth)
  - gcsfs (read GCS paths with pandas)
  - seaborn (EDA plotting)
  - pyyaml (config.yaml parsing)
  - jupyterlab (optional, for local)
- All pinned to tested versions

### README.md (updated)
**Implicit:** From M2 running instructions
- Added M2 link & Colab URL
- Purpose section (clarified binary classification task)
- M2 running instructions:
  - Prerequisites (Python, dependencies, M1 output)
  - Colab option (with auth)
  - Local Jupyter option
  - Output files location

---

## Decision Records

### Why Notebook vs. Separate Scripts?
**Prompt (implicit):** "can this be done in a jupyter notebook?"
- Answer: Yes, full M2 in single notebook
- Rationale: Peer collaboration, easy to share, mixed code/markdown

### Why One Notebook vs. Split Notebooks?
**Prompt (implicit):** "plan to create solution in a notebook called notebook-skin-2"
- Answer: Single `notebook-skin-2.ipynb` (not 5 separate notebooks)
- Rationale: Linear progression, clear narrative, easier to follow

### Why GCS Over Local Files?
**Prompt (implicit):** "plan to create solution" (spec says storage must be justified)
- Answer: GCS + Parquet chosen
- Justification in `docs/storage_design.md`:
  - Write-once, read-whole-file access pattern fits object store
  - Cost effective (<$1/month)
  - Scales to M3 distributed training
  - Direct integration with Colab/Vertex AI

### Why These Design Docs?
**Prompt 4:** "does this plan meet all the spec requirements"
- Gap identified: Need to justify storage, plan baseline model, document next steps
- Solution: Added 7 design docs covering all gaps

---

## Specification Compliance

All files created to meet **M2 Spec Requirements** (from docs/specs/m2-spec.md):

**Submission Requirements (1.1 - 1.3):**
- ✅ Milestone Report sections → Design docs (project_definition, data_source, etc.)
- ✅ Repository Evidence → README, requirements, notebook, folder structure
- ✅ Pipeline Diagram → docs/pipeline_architecture.md

**Technical Expectations (Section 2):**
- ✅ Data Acquisition → Cell 2
- ✅ Validation → Cell 4
- ✅ Cleaning & Transformation → Cells 6-7
- ✅ Data Splitting → Cell 9
- ✅ Storage → docs/storage_design.md
- ✅ Reproducibility → config.yaml, requirements.txt, README
- ✅ Evidence Quality → Cell 5 (interpreted EDA)

**Key Focus Areas (Section 3):**
- ✅ EDA (20%) → Cell 5 (purposeful, interpreted)
- ✅ Pipeline Implementation (25%) → Cells 1-10 (executable, modular)
- ✅ Reproducibility (10%) → config, requirements, README

**Grading Rubric (Section 3):**
- ✅ Project definition → project_definition.md
- ✅ Data provenance & ethics → data_source.md
- ✅ Data understanding & EDA → Cell 5 + eda_*.png
- ✅ Pipeline architecture & storage → pipeline_architecture.md + storage_design.md
- ✅ Pipeline implementation → notebook-skin-2.ipynb (Cells 1-10)
- ✅ Reproducibility & documentation → README, requirements, docs/
- ✅ Next-step plan → next_steps.md + Cell 11

---

## Summary

**Total Files Created: 11**
1. notebook-skin-2.ipynb (main deliverable)
2. config.yaml
3. project_definition.md
4. data_source.md
5. pipeline_architecture.md
6. feature_engineering.md
7. storage_design.md
8. baseline_model.md
9. next_steps.md
10. requirements.txt (updated)
11. README.md (updated)

**All generated from:** M2 spec requirements + project context (skin cancer classification task)

**Code generated:** Caveman mode (terse, substance-focused)

**All files:** Production-ready, ready for M2 submission and M3 baseline training
