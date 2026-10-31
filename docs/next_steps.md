# Next Steps: Milestone 3 Planning

## Unresolved Data Issues from M2

### 1. Class Imbalance (19.5% Malignant)

**Issue:** Highly imbalanced dataset (1 malignant : 4 benign)

**Impact:**
- Model can achieve 80.5% accuracy by predicting "benign" for everything
- Requires weighted loss or resampling to force learning

**M3 Solution:**
- Use `class_weight={0: 0.8, 1: 4.1}` in model training (see `docs/baseline_model.md`)
- Alternative: Oversampling malignant cases during training
- Monitor: Recall (sensitivity) must be ≥ 0.85 to catch malignancies

### 2. Image Quality Variance

**Issue:** Input images have varying resolution, lighting, angle

**Evidence:** EDA shows pixel value ranges 0-255; some images may be underexposed/overexposed

**Impact:**
- Model must learn robustness to variation
- May miss lesions with poor image quality

**M3 Solution:**
- Image augmentation: rotation, brightness, zoom (adds robustness)
- Normalization: Ensure consistent pixel value distribution
- Data quality filtering: Flag and possibly exclude very poor images

### 3. Missing Values Handling

**Issue:** ~8% age missing, ~2% sex missing, <1% location missing

**M2 Solution:** Median imputation for age; `unknown` category for sex/location

**M3 Validation:**
- Check that imputation doesn't bias model
- Verify dev/test distributions match train
- Consider masking features (indicate which were imputed)

## Baseline Model Specification (M3)

### Architecture

**See** `docs/baseline_model.md` for full details.

**Quick Summary:**
- Pre-trained MobileNetV2 (image feature extraction)
- Concatenate with tabular features (age, sex, location)
- 2 dense layers + sigmoid output
- Binary cross-entropy loss with class weights

**Hyperparameters (Baseline):**
- Learning rate: 1e-3 (Adam optimizer)
- Batch size: 32
- Epochs: 50 (early stopping on dev AUC)
- Dropout: [0.5, 0.3]
- Class weights: {0: 0.8, 1: 4.1}

### Training Data

- **Train:** 8,012 images + features
- **Dev:** 1,001 (validation, hyperparameter tuning)
- **Test:** 1,002 (final evaluation, locked until end)

### Evaluation Metrics

| Metric | Target | Why |
|--------|--------|-----|
| AUC-ROC | ≥ 0.85 | Overall discriminative ability; threshold-independent |
| Recall (Sensitivity) | ≥ 0.85 | Catch ≥85% of malignant cases (safety critical) |
| Precision | ≥ 0.50 | For screening tool, acceptable false positive rate |
| F1-Score | ≥ 0.65 | Balance precision & recall |

**Rationale:** Screening task prioritizes recall (don't miss malignancies); precision is secondary.

## Unresolved Model Design Decisions

### 1. Fine-tuning Strategy

**Question:** How much of MobileNetV2 to fine-tune on dermoscopy data?

**Options (M3 Tuning):**
- A) Freeze all, train only top dense layers (fast, lower accuracy)
- B) Unfreeze last 5 layers (balanced)
- C) Unfreeze last 10+ layers (slow, likely overfit on small dataset)

**Baseline Recommendation:** Option B (last 10 layers unfreeze)

**Decision Point:** Grid search on dev AUC in M3

### 2. Image Augmentation

**Question:** What augmentation to apply during training?

**Baseline (Conservative):**
- Random rotation ± 15°
- Random brightness ±20%
- Random zoom 0.8-1.2×
- **Why:** Realistic natural variation in dermoscopy acquisition

**Advanced (M4 Optional):**
- ColorJitter (mimic different camera white balance)
- Elastic deformation (simulate pressing with dermatoscope harder/softer)
- Mixup / CutMix (advanced regularization)

### 3. Handling Lesion Dependencies

**Issue:** Multiple images per lesion; all in training data

**Current:** No explicit handling (images treated independently)

**Alternatives (M3):**
- A) Current (implicit): Model learns to ignore redundancy
- B) Average predictions: Predict per image, average for lesion
- C) Explicit loss: Group loss that penalizes lesion-level errors

**Baseline:** Option A (simplest, works in practice)

**Validation:** Check that CV folds maintain lesion integrity (done in M2)

## Planned Baselines (M3)

### Baseline 1: CNN + Tabular (Primary)

**Architecture:** MobileNetV2 + dense layers (see baseline_model.md)

**Expected AUC:** 0.80-0.85

**Timeline:** 1-2 hours training on GPU

### Baseline 2: Image-Only CNN (Ablation)

**Architecture:** Same but without tabular features

**Expected AUC:** 0.78-0.83 (slightly lower)

**Purpose:** Quantify contribution of age/sex/location

### Baseline 3: Tabular-Only Classifier (Ablation)

**Architecture:** Random Forest or XGBoost on age/sex/location

**Expected AUC:** 0.55-0.60 (barely better than random)

**Purpose:** Show images dominate signal

## Evaluation Approach (M3)

### Training

1. **Fit preprocessing on train split:**
   - Image normalization params
   - Age z-score params
   - Categorical encoding

2. **Train on train split:**
   - 50 epochs (early stopping on dev AUC)
   - Monitor: loss, AUC, precision, recall

3. **Tune on dev split:**
   - Hyperparameter grid search if time allows
   - Options: learning rate, batch size, dropout

4. **Evaluate on test split (locked, no peeking):**
   - Final metrics: AUC, precision, recall, F1, confusion matrix

### Validation

**K-Fold Cross-Validation (Optional):**
- Use M1 8-fold CV splits
- Train 8 models, average metrics
- Confidence intervals for robustness

**Stratification Check:**
- Ensure class balance consistent train/dev/test
- No systematic drift

**Per-Subgroup Performance:**
- Segment by age (young/middle/old)
- Segment by sex (if enough samples)
- Segment by location (face vs. body)
- **Goal:** Identify fairness issues

## Interpretation & Reporting (M3)

### What to Report

1. **Performance Summary:**
   - AUC-ROC with 95% CI
   - Precision / Recall / F1 at optimal threshold
   - Confusion matrix (TP, TN, FP, FN)

2. **Learning Curves:**
   - Training loss vs. epoch
   - Dev AUC vs. epoch
   - Show convergence and overfitting trends

3. **Feature Importance (Approximate):**
   - Gradient-based importance: image features vs. tabular
   - Permutation importance: drop each tabular feature, see impact

4. **Failure Analysis:**
   - Show misclassified examples (images + features)
   - Patterns in errors (e.g., light skin vs. dark skin)

5. **Demographic Parity:**
   - Performance metrics by age, sex, location
   - Flag if large disparities

### Example Report Sections

**Results Summary**
```
Baseline CNN + Tabular Model

Train AUC:    0.88
Dev AUC:      0.84
Test AUC:     0.83 (95% CI: 0.81-0.85)

Test Metrics (optimal threshold):
- Precision:  0.52 (52% of predicted-malignant are actually malignant)
- Recall:     0.85 (catch 85% of malignant cases)
- F1-Score:   0.65
- Specificity: 0.89 (avoid false alarms)

Confusion Matrix (test):
         Predicted: Neg  Pos
Actual: 
Neg (0):     892    48     (specificity=0.949)
Pos (1):      15    47     (recall=0.758)
```

## M3 Timeline & Deliverables

### Schedule

**Week 1 (3-4 days):**
- Set up training environment (GPU access)
- Preprocess images (load, augment)
- Implement architecture

**Week 2 (3-4 days):**
- Train baseline CNN + tabular
- Hyperparameter tuning
- Cross-validation (if time)

**Week 3 (2-3 days):**
- Final evaluation on test split
- Interpretation & analysis
- Write M3 report

### Deliverables

**Code:**
- `notebooks/notebook-skin-3.ipynb` (training, evaluation)
- `scripts/train.py` (optional, if separating code)
- `models/baseline_cnn_tabular.h5` (trained model)

**Data:**
- `results/training_history.csv` (per-epoch metrics)
- `results/test_metrics.json` (final test performance)
- `results/confusion_matrix.png` (visualization)

**Report:**
- `Milestone3_Report_[Name].pdf` (full analysis, interpretation, plans for M4)

## Open Questions for M3

1. **Early Stopping:** Monitor val_auc or val_loss? (recommend auc)
2. **Augmentation Strength:** Light, moderate, or aggressive? (recommend moderate)
3. **Threshold Optimization:** Default 0.5 or tuned for recall/precision trade-off? (recommend tuned)
4. **Model Checkpointing:** Save best model during training? (yes, always)
5. **Ensemble:** Single model or average of 3 models? (single for baseline)

*Decide during implementation; document choices in M3 report.*

## Stretch Goals (If Time Permits)

- [ ] 8-fold cross-validation (for robustness)
- [ ] Grid search over 5-10 hyperparameter combinations
- [ ] Grad-CAM visualization (show where model looks in images)
- [ ] Feature importance (tabular contribution vs. image)
- [ ] Per-location performance analysis
- [ ] Baseline comparison (simple rule-based classifier)

---

## Summary: M3 Mission

**Goal:** Train a working baseline model that:
- ✅ Achieves AUC ≥ 0.85 on test set
- ✅ Has recall ≥ 0.85 (catches malignancies)
- ✅ Is interpretable (shows what features matter)
- ✅ Generalizes across demographics (fair performance)

**Approach:** Pre-trained CNN + tabular features, simple architecture, documented decisions

**Success:** AUC ≥ 0.85, recall ≥ 0.85, model code + results reproducible

**Next (M4+):** Ensemble, advanced architectures, domain-specific optimizations, real-world validation
