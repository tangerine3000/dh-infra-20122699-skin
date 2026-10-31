# Storage Design & Implementation Decision

## Decision: Google Cloud Storage (GCS) + Parquet

**Choice:** Continue using GCS from Milestone 1 for processed data storage

## Justification

### 1. Consistency with M1

M1 established GCS as the data store:
- Raw data: `gs://dermalens-skin/datasets/v1/raw/`
- Combined data: `gs://dermalens-skin/datasets/v1/combined_data/`
- Splits: `gs://dermalens-skin/datasets/v1/sharded/`

**M2 builds on this:**
- Reads M1 splits from GCS
- Writes processed features back to GCS
- Maintains same versioning scheme

### 2. Cost & Scalability

**GCS (Object Store):**
- Pay for what you use (per GB-month)
- No infrastructure to manage
- Scales to terabytes without configuration
- Cost: ~$0.02/GB-month

**Alternatives:**
- Cloud SQL/PostgreSQL: $15-50+/month minimum + management overhead
- BigQuery: $6.25/TB-month for storage + query costs
- Local filesystem: Works for M2 training; doesn't scale to distributed M3 training

**Winner:** GCS for academic project with limited budget

### 3. Access Pattern Fits Object Store

**Our Pattern:**
- Write metadata once (10KB)
- Write feature matrices once (parquet ~100-500 MB)
- Read full training set for model training
- No single-row lookups
- No concurrent writes

**Database Pattern (we DON'T have):**
- Frequent row-level queries ("get age where lesion_id = X")
- Concurrent writes from multiple jobs
- Complex joins across tables

**Winner:** Object store is perfect; database would be wasteful

### 4. Integration with Model Training

**Colab/Vertex AI:**
- Direct support for `gs://` paths
- Pandas: `pd.read_parquet("gs://...")` with gcsfs
- No extra authentication needed (uses project credentials)

**Local Development:**
- Still works: `pd.read_parquet("gs://...")` with gcsfs
- Alternative: download M1 splits, work locally (shown in notebook)

**Winner:** GCS integrates seamlessly with all our environments

## Parquet Format Choice

### Why Not CSV?

**CSV Problems:**
- No schema (types inferred each read)
- Large files (uncompressed, 3-5× larger)
- Slow to read (row-by-row parsing)
- No image support (binary data = encoding overhead)

**Parquet Advantages:**
- Typed schema (int, float, string, binary)
- Columnar (read only needed columns)
- Compressed (SNAPPY by default)
- Binary image support (stored as bytes)
- Supports nested structures

**Cost Impact:** 100 MB parquet ≈ 500 MB CSV
- GCS: $0.002 vs $0.010/month per file
- Download: 5s vs 25s on 1Gbps connection
- Read time: 1s vs 3s in pandas

**Winner:** Parquet is standard for ML pipelines

## Storage Organization

### M2 Output Structure

```
gs://dermalens-skin/datasets/v1/  ← From M1
├── raw/
├── combined_data/
├── sharded/
│   ├── dermalens_splits.parquet
│   ├── train.parquet              ← Loaded in M2
│   ├── dev.parquet
│   ├── test.parquet
│   └── cv_folds/
├── manifest.json
└── m2_processed/                  ← Created in M2
    ├── features_train.parquet
    ├── features_dev.parquet
    ├── features_test.parquet
    ├── image_metadata.parquet
    └── manifest.json              ← M2 parameters & versions
```

### Why Separate Folders?

- **raw/:** Original source (never modified)
- **combined_data/:** Deduplicated combined dataset
- **sharded/:** Train/dev/test splits with metadata
- **m2_processed/:** Feature matrices ready for M3 model training

Each stage is reproducible from input folder; easier to debug or recompute.

## Data Provenance & Versioning

### Manifest File

Each stage writes a manifest tracking:
- **Input:** Version of source data
- **Parameters:** Age imputation method, encoding choices
- **Output:** Row counts, class balance, feature dimensions
- **Libraries:** Pandas, scikit-learn, etc. versions (for reproducibility)

### Example M2 Manifest

```json
{
  "stage": "m2_processing",
  "version": "v1",
  "created": "2026-10-31T12:00:00Z",
  "input": "gs://dermalens-skin/datasets/v1/sharded/",
  "preprocessing": {
    "age": {
      "method": "zscore",
      "handle_missing": "median",
      "train_mean": 50.2,
      "train_std": 19.8
    },
    "sex": {"method": "onehot"},
    "localization": {"method": "onehot"},
    "image": {
      "resize_to": [224, 224],
      "normalize": "0-1"
    }
  },
  "output_counts": {
    "train": 8012,
    "dev": 1001,
    "test": 1002
  },
  "class_balance": {
    "train": 0.195,
    "dev": 0.195,
    "test": 0.195
  }
}
```

## Accessing Data

### From Google Colab (Recommended)

```python
# Authenticate
from google.colab import auth
auth.authenticate_user()

# Read directly from GCS
import pandas as pd
df = pd.read_parquet("gs://dermalens-skin/datasets/v1/sharded/train.parquet")
```

### From Local Machine

```python
# Authenticate
# gcloud auth application-default login

import pandas as pd
df = pd.read_parquet("gs://dermalens-skin/datasets/v1/sharded/train.parquet")
```

### Offline (Download First)

```bash
gsutil -m cp -r gs://dermalens-skin/datasets/v1/sharded ~/data/m1_processed
```

Then in code:
```python
df = pd.read_parquet("~/data/m1_processed/train.parquet")
```

## Scalability for M3

When training models:

**Single Machine (CPU/GPU):**
- Load full train parquet into memory
- Fits in RAM (~500 MB)
- Standard training loop

**Distributed (Vertex AI / TPU):**
- Each job reads its shard from GCS
- No shared file system needed
- Horizontal scaling easy

**Winner:** GCS enables both single-machine and distributed training

## Security & Access Control

### IAM Permissions

**Required:**
- Read: `roles/storage.objectViewer`
- Write: `roles/storage.objectAdmin` (or `objectCreator` + `objectViewer`)

**In Practice:**
- Student account: Use Application Default Credentials
- Service account (production): Attach IAM roles to service account

### No Secrets in Data

- Data parquet files contain **no credentials, API keys, or PII**
- Safe to commit GCS paths to GitHub
- Images are dermoscopic (not identifying)
- Metadata anonymized (age, location only)

## Cost Estimate

**M1 Data:**
- Raw: ~3.7 GB
- Combined: ~1.8 GB
- Sharded: ~1.8 GB
- **Total: ~7.3 GB**

**M2 Output:**
- Features: ~50 MB (metadata only; images stay in M1)
- Metadata: ~1 MB
- **Total: ~50 MB**

**Monthly Cost (hypothetical):**
- Storage: ~$0.15 (7.3 GB × $0.02/GB-month)
- Egress (download): ~$0.01 per 1 GB downloaded
- **Total: <$1/month** (if not downloading frequently)

For academic projects: negligible cost

## Decision Summary

| Factor | Choice | Alternative | Why |
|--------|--------|-------------|-----|
| Storage Backend | GCS | Local / S3 / Azure | Consistency with M1; cost-effective; Colab integration |
| Data Format | Parquet | CSV / HDF5 / Pickle | Typed schema; compression; binary support; portable |
| Processing | CPU (Pandas) | GPU | M2 is data understanding; GPU not needed |
| Access Pattern | Read M1 splits → Write processed | Direct mutation | Preserves audit trail; enables rollback |

**M2 will output M2-processed/** with final feature matrices ready for M3 model training.
