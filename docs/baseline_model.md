# Baseline Model Plan for Milestone 3

## Architecture Decision: CNN + Tabular Fusion

### Why This Architecture?

**Rationale:**
1. **Images dominate:** Dermoscopic images contain most signal
2. **Context matters:** Patient age, sex, location improve accuracy
3. **Fusion is simple:** Concatenate embeddings; train end-to-end

**Alternatives Considered:**
- Image-only CNN: Simpler, but loses clinical features; lower accuracy
- Separate classifiers: Easier to explain, but suboptimal (no interaction learning)
- Attention mechanism: More complex; marginal gains for M3 baseline

### Proposed Architecture

```
┌──────────────────────────────┐
│   Image (224×224×3)          │
└──────────────────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Pre-trained CNN             │
│  (MobileNetV2, no top)        │
│  Output: 1280-dim embeddings  │
└──────────────────────────────┘
           │
           ├─────────────────┐
           │                 │
           ↓                 ↓
     [1280-dim]         [Age (1) + Sex (3) + Location (15)]
      Image Emb         = 19-dim tabular features
           │                 │
           ├─────────────────┤
           │
           ↓
┌──────────────────────────────┐
│  Concatenate Embeddings      │
│  [1280 + 19] = 1299-dim      │
└──────────────────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Dense Layer (256 units)     │
│  ReLU activation             │
│  Dropout (0.5)               │
└──────────────────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Dense Layer (64 units)      │
│  ReLU activation             │
│  Dropout (0.3)               │
└──────────────────────────────┘
           │
           ↓
┌──────────────────────────────┐
│  Output Layer (1 unit)       │
│  Sigmoid activation          │
│  Output: P(malignant)        │
└──────────────────────────────┘
```

## Component Details

### Image Backbone: MobileNetV2

**Choice:** Pre-trained MobileNetV2 (ImageNet weights)

**Why MobileNetV2?**
- Lightweight: 3.5M parameters (fast training)
- Pre-trained: Learns generic image features from ImageNet
- Dermoscopy transfer learning: Similar to natural images (RGB, textures)
- Deployment-ready: Small model size for mobile/embedded use

**Alternatives:**
- ResNet50: Larger, slower; ~1% better accuracy (not worth it for baseline)
- EfficientNet: Between MobileNet and ResNet; good but overkill for baseline
- VGG16: Too large; outdated

**Hyperparameters:**
- Input: 224×224 (MobileNetV2 standard)
- Weights: ImageNet pre-trained
- Fine-tuning: Unfreeze last 10 layers (tune on domain data)
- Output: 1280-dim feature vector (remove classification head)

### Tabular Fusion

**Method:** Concatenate image embeddings + encoded tabular features

**Alternative Fusion Methods:**
- Early fusion (image + features in CNN): Less effective; features overshadowed
- Late fusion (train separate models): Complex; no feature interaction
- Attention: Adaptive weighting; overkill for baseline

**Rationale for Concatenation:**
- Simple, interpretable
- Allows backprop through both streams
- Standard in multimodal ML

### Output & Loss

**Task:** Binary classification (P(malignant) ∈ [0,1])

**Loss Function:** Binary Cross-Entropy (BCE) + Class Weights

```
loss = -[y*log(p) + (1-y)*log(1-p)] * weight[y]
```

Where:
- `y` = true label (0 or 1)
- `p` = predicted probability
- `weight[1]` = cost of misclassifying malignant (higher penalty)
- `weight[0]` = cost of misclassifying benign

**Why Class Weights?**
- Dataset is ~19.5% malignant, 80.5% benign
- Without weights, model predicts "benign" for everything (high accuracy, low sensitivity)
- Weights force model to learn malignant cases despite rarity

**Typical Weights:**
- For 19.5% malignant: `weight[1] = 0.805 / 0.195 ≈ 4.1`
- Interpretation: False negative costs 4× more than false positive

## Training Strategy

### Data & Metrics

**Data:**
- Train: 8,012 images + features
- Dev: 1,001 (validation, hyperparameter tuning)
- Test: 1,002 (final evaluation)

**Metrics:**
1. **AUC-ROC:** Overall discriminative ability (target ≥ 0.85)
   - Threshold-independent
   - Good for imbalanced classes
   
2. **Precision:** TP / (TP + FP) — how many predicted-malignant are actually malignant?
   - High precision: safe to recommend biopsy

3. **Recall (Sensitivity):** TP / (TP + FN) — how many malignant are caught?
   - High recall: don't miss malignancies (critical for safety)

4. **F1-Score:** Harmonic mean of precision & recall
   - Balances both concerns

5. **Confusion Matrix:** TP, TN, FP, FN (per-class performance)

**Target Metrics (M3):**
- AUC-ROC ≥ 0.85
- Recall ≥ 0.85 (catch ≥85% of malignant)
- Precision ≥ 0.50 (acceptable false positive rate for screening)

### Training Loop

```python
model.compile(
    optimizer=Adam(learning_rate=1e-3),
    loss=BinaryCrossentropy(from_logits=False),
    metrics=['AUC', 'Precision', 'Recall']
)

# Early stopping on dev AUC
callback = EarlyStopping(
    monitor='val_auc',
    patience=5,
    restore_best_weights=True
)

history = model.fit(
    train_dataset,
    validation_data=dev_dataset,
    epochs=50,
    callbacks=[callback],
    class_weight={0: 0.8, 1: 4.1}  # Handle imbalance
)

# Final evaluation on test
test_metrics = model.evaluate(test_dataset)
```

### Hyperparameters (M3 Tuning)

| Parameter | Baseline | Range | Tuning Method |
|-----------|----------|-------|---------------|
| Learning Rate | 1e-3 | [1e-4, 1e-2] | Grid/Random |
| Batch Size | 32 | [16, 32, 64] | Grid |
| Dense Units | [256, 64] | [128-512] | Random |
| Dropout Rate | [0.5, 0.3] | [0.2-0.7] | Grid |
| Fine-tune Layers | Last 10 | [5, 10, 15] | Grid |

## Evaluation Plan

### Cross-Validation (Optional)

If resources allow:
- 8-fold CV from M1 (pre-computed in sharded/)
- Average metrics across folds
- Confidence intervals for robustness

### Per-Subgroup Analysis

For fairness & generalization:
- Performance by age group (young, middle, old)
- Performance by sex (if sample size allows)
- Performance by location (face vs. body)
- **Goal:** Ensure model works across demographics

### Failure Analysis

When model makes errors:
- Visualize misclassified images
- Analyze feature importance
- Identify systematic failure modes
- Plan improvements for M4 (optional)

## Success Criteria

**Minimum (Pass):**
- AUC-ROC ≥ 0.75
- Code runs without errors
- Metrics logged on train/dev/test

**Good (80+%):**
- AUC-ROC ≥ 0.82
- Recall ≥ 0.80 (catch most malignancies)
- Feature importance analysis
- Per-subgroup performance reported

**Excellent (90+%):**
- AUC-ROC ≥ 0.85
- Recall ≥ 0.85
- Precision ≥ 0.50
- Fair performance across demographics
- Interpretable feature importance

## Output from M3

**Deliverables:**
1. **Trained Model:** `models/baseline_cnn_tabular.h5` (TensorFlow/Keras)
2. **Training History:** `results/training_history.csv` (loss, metrics per epoch)
3. **Test Metrics:** `results/test_metrics.json` (AUC, precision, recall, F1)
4. **Confusion Matrix:** `results/confusion_matrix.png`
5. **Feature Importance:** `results/feature_importance.png` (learned embeddings / tabular impact)
6. **Per-Subgroup Analysis:** `results/subgroup_performance.csv`
7. **Model Report:** PDF with architecture, hyperparameters, results, interpretation

## Known Limitations & Future Work

### M3 Limitations

1. **Moderate AUC:** CNN + tabular unlikely to reach > 0.90 AUC
   - Optimization: ensemble, better augmentation, domain-specific architectures

2. **Limited Demographic Coverage:** Training data skews toward light skin tones
   - Mitigation: Validate carefully; flag for real-world use; note in report

3. **No Augmentation:** M2 doesn't augment images
   - M3 Improvement: Add rotation, brightness, zoom augmentation

4. **No Attention / Explainability:** Concatenation is opaque
   - M4 Improvement: Grad-CAM visualization, attention mechanisms

### M4 (Future)

- Ensemble methods (multiple models)
- Domain-specific augmentation
- Grad-CAM for image explanations
- Real-world validation on new data
- Commercial deployment considerations

---

## Summary

**Baseline Model Approach for M3:**
1. MobileNetV2 (pre-trained) for images
2. Tabular fusion (concatenate age/sex/location)
3. Dense layers + sigmoid output
4. BCE loss with class weights
5. Adam optimizer, early stopping on dev AUC
6. Target AUC ≥ 0.85, Recall ≥ 0.85

**This is a realistic, reproducible baseline.** More sophisticated methods (ensemble, attention, meta-learning) can improve on this in M4.
