# Milestone 2 Peer Grading Rubric

Milestone 2 contributes 10% of the overall module grade. This rubric records performance out of 100 for clear feedback and is then scaled to the 10% module weighting.

## Grading rubric

| Criterion | Wt. | Excellent 70–100% | Good 60–69% | Developing 40–59% | Insufficient 0–39% |
|---|---:|---|---|---|---|
| Project definition and development | 5 | Precise task, users and success criteria; changes from Milestone 1 are justified. | Task is clear with minor omissions. | Task is broadly stated but lacks precision or justification. | Unclear, inconsistent or absent project definition. |
| Data provenance, governance and ethics | 15 | Ownership, licence, collection context, privacy, bias and limitations are critically evaluated. | Source and licence are clear; main governance and ethical issues are discussed. | Source is identified but provenance, licence or risks are treated superficially. | Source/permission is unclear, or material ethical and legal risks are ignored. |
| Data understanding and EDA | 20 | Accurate, purposeful analysis of structure, quality and distributions; visualisations are interpreted and drive decisions. | Sound EDA covers the main quality and distribution issues with reasonable interpretation. | Basic summaries are present but incomplete, weakly interpreted or disconnected from decisions. | EDA is absent, incorrect, largely automated without interpretation, or not relevant to the task. |
| Pipeline architecture and storage design | 15 | Coherent end-to-end design; storage and processing choices are technically justified and match the project constraints. | Clear pipeline with mostly appropriate choices and some justification. | Partial or generic design; important stages or justifications are missing. | No credible pipeline design or inappropriate, unexplained architecture. |
| Pipeline implementation | 25 | Working, modular pipeline with acquisition, validation and transformation; handles failures and produces repeatable outputs. | Core pipeline runs and implements the main stages; some robustness or modularity is missing. | Partially working pipeline or notebook with substantial manual steps and limited validation. | Little executable evidence, major failures, or code does not correspond to the claimed process. |
| Reproducibility, repository and documentation | 10 | Another user can set up and run the work; dependencies, structure, commits and instructions are clear; secrets/data are managed correctly. | Repository is organised and mostly reproducible with minor documentation gaps. | Repository is accessible but setup or execution requires unstated steps. | Repository is inaccessible, disorganised, unsafe or not reproducible. |
| Next-step plan | 10 | Limitations and proposed baseline, evaluation approach and Milestone 3 plan. | Main limitations and next steps are identified and feasible. | General reflection with limited prioritisation or technical detail. | No credible reflection or plan for subsequent work. |

## Mark calculation

Each criterion receives a percentage judgement within the descriptor band. The weighted criterion marks produce a Milestone 2 score out of 100. This score is multiplied by 0.10 to calculate the contribution to the overall module grade.

Formula:

- Milestone 2 score (/100) = sum of each criterion mark
- Module contribution = Milestone 2 score × 0.10

## Reviewer guidance

- Assign a percentage judgement for each criterion using the descriptor that best matches the work.
- Where a project sits between two bands, use the most appropriate midpoint judgement.
- The most important evidence should come from the repository contents, notebook outputs and accompanying documentation.
- Use the criterion weighting to convert the final judgement into the overall Milestone 2 grade.

## Suggested reviewer checklist

- Is the project scope clear and justified?
- Are data provenance and ethics discussed with appropriate depth?
- Does the EDA meaningfully explain data quality and distributions?
- Is the pipeline architecture justified against the project constraints?
- Is the implementation working and reproducible?
- Are the next steps realistic and clearly connected to Milestone 3?

---

## Applied evaluation for this project

This repository was evaluated against the rubric using the project documentation and notebooks as evidence.

| Criterion | Weight | Score awarded | Evidence |
|---|---:|---:|---|
| Project definition and development | 5 | 82/100 | The task, scope, and milestone structure are clearly defined in [README.md](README.md), [notebook-skin.ipynb](notebook-skin.ipynb), and [notebook-skin-2.ipynb](notebook-skin-2.ipynb). The project now includes a clear end-user requirement section in [README.md](README.md). |
| Data provenance, governance and ethics | 15 | 90/100 | Source ownership, licensing, privacy, bias, and risk are documented in [docs/governance_and_licensing.md](docs/governance_and_licensing.md) and [docs/data_source.md](docs/data_source.md). |
| Data understanding and EDA | 20 | 89/100 | Purposeful EDA is included in [notebook-skin-2.ipynb](notebook-skin-2.ipynb), covering class balance, age, sex, and lesion-location analysis with interpretation. |
| Pipeline architecture and storage design | 15 | 85/100 | Storage and processing rationale is described in [notebook-skin.ipynb](notebook-skin.ipynb) and [config.yaml](config.yaml), including GCS, Parquet, and versioned data layout. |
| Pipeline implementation | 25 | 80/100 | The notebook implements validation, preprocessing, and feature engineering in [notebook-skin-2.ipynb](notebook-skin-2.ipynb). Explicit checks for required fields and leakage are present. |
| Reproducibility, repository and documentation | 10 | 88/100 | Setup and reproduction guidance are provided in [README.md](README.md), [docs/reproducibility.md](docs/reproducibility.md), and [requirements.txt](requirements.txt). |
| Next-step plan | 10 | 84/100 | The project identifies class imbalance, augmentation, and missing-value handling as next steps in [notebook-skin-2.ipynb](notebook-skin-2.ipynb). |

### Weighted calculation

- Project definition: 5 × 0.82 = 4.10
- Data provenance: 15 × 0.90 = 13.50
- Data understanding and EDA: 20 × 0.89 = 17.80
- Pipeline architecture: 15 × 0.85 = 12.75
- Pipeline implementation: 25 × 0.80 = 20.00
- Reproducibility: 10 × 0.88 = 8.80
- Next-step plan: 10 × 0.84 = 8.40

Total score = 85.35 / 100

Module contribution = 85.35 × 0.10 = 8.54%

### Reviewer summary

This project is a strong Milestone 2 submission. It demonstrates clear project definition, well-developed governance and ethics documentation, strong EDA with interpretation, and a credible storage and pipeline design. The repository is also considerably more reproducible and easier to understand than earlier iterations. The main remaining weakness is that the implementation would benefit from a formal end-to-end validation log or run artifact to demonstrate a complete execution record in the repository.

This rubric should be used as a peer-review tool for the Milestone 2 submission in the Dermalens Skin Cancer Classification Pipeline project.
