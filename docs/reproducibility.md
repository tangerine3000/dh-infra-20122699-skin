# Reproducibility Guide

## Environment setup

1. Create a virtual environment.
   ```bash
   python -m venv .venv
   source .venv/bin/activate  # macOS/Linux
   .venv\Scripts\activate     # Windows
   ```

2. Install pinned project dependencies.
   ```bash
   pip install -r requirements.txt
   ```

3. Confirm the local environment matches the notebook expectations.
   ```bash
   python -m ipykernel install --user --name dermalens-env
   ```

## Dataset and storage expectations

- The repository expects either:
  - a working Google Cloud Storage path, or
  - a local dataset folder matching the data structure defined in `config.yaml`.
- Use the same `gcs_bucket` and `gcs_prefix` values recorded in the configuration file.
- Do not overwrite raw or versioned data folders without a deliberate version bump.

## Reproducibility checklist

Before sharing or running a final notebook:
- confirm the dataset revision is still the intended source revision;
- verify the notebook is run from the repository root;
- check that `config.yaml` points to the correct paths;
- confirm the environment uses the pinned dependency versions;
- validate that train/dev/test outputs are generated from the same dataset version; and
- record any deviations in the notebook output or README notes.

## Recommended practice

- Keep an explicit note of the source dataset revision in the notebook markdown.
- Save generated artifacts under versioned directories such as `data/processed/` or `datasets/v1/`.
- Prefer `git status` and commit history to track reproducibility changes.
- Treat raw data as immutable and create new processed outputs when the pipeline changes.
