# Why CV Fold Shards Are Kept Separate

## Purpose of the separate fold shards

The project keeps its cross-validation (CV) fold shards as separate files rather than storing a single combined training set because this preserves a clean and reproducible evaluation workflow.

Each fold is generated from the training data only, and each validation fold contains a disjoint subset of the training lesions. This avoids information leakage between training and validation partitions and makes every fold independently auditable.

## Why this matters

The main risk in medical imaging projects is lesion-level leakage. A single lesion may have multiple dermoscopic images, and those images should not be split across train and validation sets. If several images from the same lesion appear in both places, the model can effectively memorise lesion-specific patterns rather than learning generalisable features.

Keeping folds separate ensures:

- no lesion-level leakage across folds;
- reproducible validation results;
- easier debugging when a fold fails or underperforms;
- a clear record of how the model was evaluated;
- a training setup that can work with large parquet files without loading unnecessary data.

## How the project implements this

The configuration in `config.yaml` explicitly sets:

- `group_by_lesion_id: true`
- `strategy: stratified_grouped`

This means the split is grouped by `lesion_id` and stratified by the target label. The training data is then partitioned into multiple folds, and each fold stores its own train/validation shard files, for example:

- `cv_folds/fold_0_train.parquet`
- `cv_folds/fold_0_val.parquet`
- `cv_folds/fold_1_train.parquet`
- `cv_folds/fold_1_val.parquet`

These shard files are kept separate so the training pipeline can read only the fold it needs, while preserving the exact holdout split for that fold.

## Why this is better than a single combined file

A combined file would make it harder to:

- enforce strict train/validation separation;
- recover the exact fold assignment used during evaluation;
- repeat a training run with the same data split;
- review whether a model was trained and validated correctly.

Using separate fold shards makes the workflow easier to audit and more consistent with best practice in medical ML pipelines.

## Link to the project implementation

This design is reflected in the project notebook and storage documentation, including:

- [notebook-skin.ipynb](../notebook-skin.ipynb)
- [config.yaml](../config.yaml)
- [docs/pipeline_architecture.md](pipeline_architecture.md)
