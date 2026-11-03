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

