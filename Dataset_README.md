---
license: cc-by-nc-sa-4.0
task_categories:
- image-classification
tags:
- dermatology
- skin-cancer
- medical-imaging
- HAM10000
pretty_name: DermaLens Skin Cancer Dataset
size_categories:
- 10K<n<100K
configs:
- config_name: default
  data_files:
  - split: train
    path: data/train-*
  - split: validation
    path: data/validation-*
  - split: test
    path: data/test-*
dataset_info:
  features:
  - name: image
    dtype: image
  - name: image_id
    dtype: string
  - name: lesion_id
    dtype: string
  - name: dx
    dtype: string
  - name: dx_type
    dtype: string
  - name: age
    dtype: float64
  - name: sex
    dtype: string
  - name: localization
    dtype: string
  splits:
  - name: train
    num_bytes: 2650122851
    num_examples: 9577
  - name: validation
    num_bytes: 689647139
    num_examples: 2492
  - name: test
    num_bytes: 354617559
    num_examples: 1285
  download_size: 3693404553
  dataset_size: 3694387549
---

# DermaLens Skin Cancer Dataset

This dataset repo documents the data pipeline used to train the **DermaLens V3** skin cancer classification model.

## Source Dataset

**HAM10000** (Human Against Machine with 10000 training images) — accessed via [`marmal88/skin_cancer`](https://huggingface.co/datasets/marmal88/skin_cancer) on HuggingFace.

```python
from datasets import load_dataset
ds = load_dataset("marmal88/skin_cancer")
```

## Dataset Statistics

| Split | Images | Malignant | Benign | Positive Rate |
|-------|--------|-----------|--------|---------------|
| Train | 10,683 | ~2,093 | ~8,590 | 19.6% |
| Validation | 1,335 | ~263 | ~1,072 | 19.7% |
| Test | 1,336 | ~253 | ~1,083 | 18.9% |
| **Total** | **13,354** | **~2,609** | **~10,745** | **19.5%** |

## Label Mapping (Binary)

| Original Class (`dx`) | Binary Label | Category |
|------------------------|-------------|----------|
| `melanoma` | 1 (Malignant) | Malignant melanocytic |
| `basal_cell_carcinoma` | 1 (Malignant) | Non-melanocytic malignant |
| `actinic_keratoses` | 1 (Malignant) | Pre-cancerous |
| `melanocytic_Nevi` | 0 (Benign) | Common moles |
| `benign_keratosis-like_lesions` | 0 (Benign) | Seborrheic keratoses etc. |
| `dermatofibroma` | 0 (Benign) | Benign fibrous |
| `vascular_lesions` | 0 (Benign) | Angiomas etc. |

```python
MALIGNANT_CLASSES = {"melanoma", "basal_cell_carcinoma", "actinic_keratoses"}
label = 1 if item["dx"] in MALIGNANT_CLASSES else 0
```

## Preprocessing

- **Resize**: 384x384 pixels
- **Normalization**: ImageNet stats (mean=[0.485, 0.456, 0.406], std=[0.229, 0.224, 0.225])
- **Augmentation** (train only): RandomResizedCrop, Flips, Rotation, ColorJitter, CoarseDropout, CLAHE
- **Oversampling**: Malignant samples repeated 3x

## Model Performance (DermaLens V3)

| Metric | Value |
|--------|-------|
| Test ROC-AUC | **0.9753** |
| Test PR-AUC | **0.9127** |
| Test F1 | **0.8457** |
| Sensitivity | 94% |
| Specificity | 91% |

Model weights: [`dheraingoud/dermalens-model`](https://huggingface.co/dheraingoud/dermalens-model)
