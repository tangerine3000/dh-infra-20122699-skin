# Data Source & Governance

## Source Information

**Dataset:** Dermalens  
**Platform:** Hugging Face Datasets  
**URL:** https://huggingface.co/datasets/dheraingoud/dermalens-datasets  
**Original Source:** HAM10000 database  
**Pinned Revision:** 52bb1787931b301d610829724dc78c15c2cab797  

## Data Ownership & Permissions

**Original Data Owner:** HAM10000 project (University of Queensland)

**Licensing:**
- HAM10000 dataset: CC-BY-NC 4.0 (Creative Commons Attribution-NonCommercial)
- **Restriction:** NonCommercial use only. This project is for academic/educational purposes.

**Access:** Publicly available through Hugging Face; no credentials required for download.

## Collection Context

**Source Study:**
- Dermoscopic imaging database of skin lesions
- Multi-institutional collaboration
- Diverse patient demographics

**Data Characteristics:**
- Dermoscopic images (high-quality, magnified views)
- Associated metadata: diagnosis, confirmation method, patient age, sex, lesion location
- Pre-split into train/validation/test by source

**Image Properties:**
- Format: JPG/PNG
- Resolution: Variable (typically 256×256 to 1024×1024 pixels)
- Color space: RGB
- Total images: 13,354 (before deduplication)

## Privacy & Ethical Considerations

### Privacy

**Anonymization:**
- Patient names and personal identifiers are not included
- Images are dermoscopic (not full-body photography)
- Only age and sex are retained; no other PII

**Data Usage Restrictions:**
- Non-commercial use only (CC-BY-NC 4.0)
- Academic projects permitted
- Must cite original HAM10000 source

**Sensitive Information:**
- Patient age (de-identified)
- Lesion anatomical location (not necessarily sensitive)

### Ethical Considerations

**Demographic Bias:**
- Dataset may reflect demographic characteristics of dermatology clinics
- Potential underrepresentation of darker skin tones (known issue in dermoscopy datasets)
- **Mitigation:** Results must not be deployed without validating performance across skin tones
- **Documentation:** Analyze class balance and performance stratification by demographic factors

**Clinical Use:**
- System is a screening aid, not a diagnostic tool
- High false-negative rate (missed malignancies) is unacceptable
- Requires clinical validation before real-world deployment
- Should not replace expert clinical judgment

**Potential Misuse:**
- Model should not be used for autonomous diagnosis without human review
- Risk of algorithmic bias in resource-constrained settings where model is used without oversight

## Data Quality & Limitations

### Known Limitations

1. **Class Imbalance:** ~19.5% malignant, ~80.5% benign
   - Reflects real-world prevalence but requires careful handling

2. **Image Quality Variance:** Differing acquisition conditions, magnification, lighting
   - Some images may have artifacts or poor quality

3. **Missing Values:** 
   - Age missing in ~8% of records
   - Sex missing in ~2% of records
   - Location missing in <1% of records

4. **Demographic Representation:** 
   - Primarily Caucasian/light skin tones
   - Limited diversity in age (underrepresentation of very young and very old)

5. **Temporal:** No temporal information; snapshot diagnosis from time of acquisition

6. **Multi-Image Lesions:** Some lesions have multiple images
   - Milestone 1 splits preserve lesion integrity to prevent leakage
   - Milestone 2 EDA must account for this dependency

### Quality Assurance (Milestone 1)

- Deduplication: 3,339 exact duplicates removed
- Rows retained: 10,015
- Stratified split preserves class balance across train/dev/test

## Data Storage & Access

**Milestone 1 Storage:**
- Google Cloud Storage (GCS)
- Bucket: `gs://dermalens-skin`
- Prefix: `datasets/v1/`
- Format: Apache Parquet (columnar, typed, compressed)
- Access: Pandas `read_parquet("gs://...")` with `gcsfs`

**File Structure:**
```
gs://dermalens-skin/datasets/v1/
├── raw/                          (original HF parquet splits)
├── combined_data/                (deduplicated combined dataset)
├── sharded/                       (train/dev/test partitions)
│   ├── train.parquet
│   ├── dev.parquet
│   ├── test.parquet
│   └── cv_folds/                 (8 cross-validation splits)
└── manifest.json                 (metadata, versions, counts)
```

## Reproducibility

**To Reproduce:**
1. Run Milestone 1 notebook (`notebook-skin.ipynb`)
2. Pinned HF revision ensures identical source data
3. Seed=42 ensures identical splits
4. Manifest includes all parameters and library versions

**Citation:**
```
@dataset{dermalens,
  title={Dermalens Datasets},
  url={https://huggingface.co/datasets/dheraingoud/dermalens-datasets},
  year={2024}
}

Original source:
@article{ham10000,
  title={HAM10000 dataset: A large collection of multi-source dermatoscopic images},
  author={Tschandl et al.},
  journal={Scientific Data},
  year={2018}
}
```

## Compliance

- **Licence:** CC-BY-NC 4.0 (attribution required, non-commercial)
- **Academic Use:** Permitted
- **Clinical Deployment:** Not permitted without commercial licence negotiation
