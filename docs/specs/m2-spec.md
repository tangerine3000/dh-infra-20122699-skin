# Milestone 2 Specification: Data Understanding and Pipeline Setup

**Module:** Data Handling and Infrastructure for AI  
**Deadline:** Sunday, 1st November 2026 at 23:59  
**Submission:** Moodle (one PDF report + repository link)  
**Weightage:** 10% of overall module grade  
**Assessment Focus:** Initial report on data and reproducible data pipeline setup

---

## Overview

Milestone 2 demonstrates that you understand your selected data and can move it through a documented, repeatable pipeline. **A final model is not required at this stage.**

### Key Principle: Carry Forward, Do Not Duplicate
- You may use Milestone 1 as the foundation for this report
- If project definition, data-split strategy, or storage decision is unchanged, **summarise or reference it briefly**
- Marks are based on **new evidence**: EDA, implementation, validation, integration, and justified changes

---

## What You Must Complete

- [ ] A confirmed project question and clearly defined analytical, prediction, classification, or forecasting task
- [ ] A justified dataset, including origin, licence, structure, limitations, and ethical implications
- [ ] **Purposeful EDA** that is interpreted in your own words and informs data-quality, preprocessing, splitting, and pipeline decisions
- [ ] A reproducible process for acquiring, storing, validating, cleaning, and transforming data
- [ ] A documented repository that another student could set up and run from your instructions

---

## Learning Outcomes Addressed

This milestone provides evidence for:
- Data comparison and feature engineering
- Data-pipeline design and implementation
- Evaluation of suitable data-storage approaches

---

# 1. Required Submission

## 1.1 Milestone Report (PDF)

Submit one **self-contained PDF report** (excluding references and appendices).

### Required Sections

| Section | Contents |
|---------|----------|
| **Project definition** | Current title, problem statement, intended users/setting, exact analytical or ML task. Explain any change from Milestone 1. |
| **Data source** | Source URL or DOI, data owner, licence/terms of use, collection context, permissions, privacy, bias, and ethical considerations. |
| **Data description** | Unit of analysis, number of records, fields, data types, target variable (if applicable), time range, class balance, missingness, and known limitations. |
| **Exploratory Data Analysis** | **Central focus of Milestone 2.** Include relevant statistics and purposeful visualisations. **Interpret every result** and explain how it affects data quality, cleaning, transformation, feature engineering, splitting, or evaluation. |
| **Design/Architecture** | A diagram showing: acquisition → ingestion → validation → storage → processing → output. State what is automated and what remains manual. |
| **Implementation** | Evidence that the pipeline runs on a representative sample or full dataset. Include commands, screenshots, or concise outputs. If storage/splitting implemented early, demonstrate integration and validation rather than repeating original justification. |
| **Next Steps** *(Optional)* | Unresolved data issues, intended baseline model, and plan for Milestone 3. |

---

## 1.2 Repository Evidence

Your repository must include:

- **README.md**
  - Project purpose
  - Data source
  - Environment setup instructions
  - **Exact run instructions** (step-by-step reproducibility)

- **Requirements/Environment file**
  - Pinned or clearly stated dependencies
  - Specify versions where critical

- **Scripts/Notebooks** for:
  - Data acquisition
  - Validation
  - Cleaning
  - Transformation
  - EDA

- **Clear folder structure**
  - Logical organisation that a new user can navigate
  - ❌ Do NOT commit: credentials, API keys, personal data, unnecessarily large raw datasets

- **Meaningful commits**
  - Show development over time
  - Clear commit messages
  - Submitted link must be **accessible to the lecturing team**

---

## 1.3 Pipeline Diagram (Optional but Recommended)

Include one **readable architecture or data-flow diagram**.

**Minimum requirement — show:**
```
Source → Ingest → Validate → Store → Transform → Output
```

**Critical:** The diagram must correspond to your **implemented code and storage choices**. A generic cloud diagram without connection to your project will **not receive full credit**.

---

# 2. Technical Expectations

### Data Acquisition
- Data can be downloaded, queried, or loaded using a **documented command, script, or notebook**
- If access is manual, explain why and preserve a **reproducible local entry point**
- Example: `python scripts/fetch_data.py` or notebook with clear instructions

### Validation
- Check: schema, data types, required fields, duplicates, valid ranges, obvious inconsistencies
- Failures should be **visible** (logged/reported), not silently ignored
- Implement assertions or explicit validation functions
- Example: schema validation, type checking, constraint verification

### Cleaning and Transformation
- **Justified handling** of:
  - Missing values (explain imputation/removal strategy)
  - Duplicates (detection and removal logic)
  - Categorical variables (encoding, binning decisions)
  - Scaling/normalisation (when and why)
  - Text/image preparation (if applicable)
  - Dates (parsing, standardisation)
  - Outliers (detection and treatment rationale)
- Document all decisions with reasoning

### Data Splitting
- If strategy is unchanged from Milestone 1, **summarise briefly**
- Show **how it is implemented** or **validated**
- Explain any changes and rationale
- Use appropriate splitting strategy: chronological, grouped, or stratified (justify choice)

### Storage
- If choice is unchanged from Milestone 1, **summarise or reference it**
- Provide **evidence that storage is:**
  - Implemented and functional
  - Integrated with the pipeline
  - Appropriate to scale, format, and access pattern
- ℹ️ Cloud storage is **not mandatory** for its own sake; choose based on actual requirements

### Reproducibility
- A **clean environment** should be able to run the documented pipeline without:
  - Hidden steps
  - Private file paths
  - Embedded secrets
  - Undocumented dependencies
- Test this: clone your repo in a fresh environment and verify it works

### Evidence Quality
- Figures and tables must be **legible, labelled, and interpreted**
- ❌ Large raw notebook dumps do **not** substitute for analysis
- Show insights, not just outputs

---

# 3. Grading Rubric

**Milestone 2 contributes 10% of overall module grade.**  
Rubric records performance out of 100, then scaled to 10% weighting.

| Criterion | Wt. | Excellent (70–100%) | Good (60–69%) | Developing (40–59%) | Insufficient (0–39%) |
|-----------|-----|-----|-----|-----|-----|
| **Project definition and development** | 5 | Precise task, users, and success criteria; changes from M1 are justified. | Task is clear with minor omissions. | Task is broadly stated but lacks precision or justification. | Unclear, inconsistent, or absent project definition. |
| **Data provenance, governance, and ethics** | 15 | Ownership, licence, collection context, privacy, bias, and limitations are critically evaluated. | Source and licence are clear; main governance and ethical issues are discussed. | Source is identified but provenance, licence, or risks are treated superficially. | Source/permission unclear; material ethical/legal risks ignored. |
| **Data understanding and EDA** | 20 | Accurate, purposeful analysis of structure, quality, and distributions; visualisations are interpreted and drive decisions. | Sound EDA covers main quality and distribution issues with reasonable interpretation. | Basic summaries present but incomplete, weakly interpreted, or disconnected from decisions. | EDA absent, incorrect, largely automated without interpretation, or irrelevant to task. |
| **Pipeline architecture and storage design** | 15 | Coherent end-to-end design; storage and processing choices are technically justified and match project constraints. | Clear pipeline with mostly appropriate choices and some justification. | Partial or generic design; important stages or justifications missing. | No credible pipeline design or inappropriate, unexplained architecture. |
| **Pipeline implementation** | 25 | Working, modular pipeline with acquisition, validation, and transformation; handles failures and produces repeatable outputs. | Core pipeline runs and implements main stages; some robustness or modularity missing. | Partially working pipeline or notebook with substantial manual steps and limited validation. | Little executable evidence, major failures, or code does not correspond to claimed process. |
| **Reproducibility, repository, and documentation** | 10 | Another user can set up and run the work; dependencies, structure, commits, and instructions are clear; secrets/data managed correctly. | Repository is organised and mostly reproducible with minor documentation gaps. | Repository is accessible but setup/execution requires unstated steps. | Repository inaccessible, disorganised, unsafe, or not reproducible. |
| **Next-step plan** | 10 | Limitations and proposed baseline, evaluation approach, and M3 plan are clear. | Main limitations and next steps identified and feasible. | General reflection with limited prioritisation or technical detail. | No credible reflection or plan for subsequent work. |

**Mark Calculation:**
- Each criterion receives a percentage judgement within the descriptor band
- Weighted criterion marks produce a Milestone 2 score out of 100
- Score is multiplied by 0.10 to calculate contribution to overall module grade
- *Example: 72/100 contributes 7.2 marks out of available 10*

---

## Key Focus Areas for High Marks

### 1. **EDA (20% weight) — This is the central focus**
- Go beyond summary statistics
- Create purposeful visualisations (not generic plots)
- **Interpret every finding** in plain language
- **Explain how each finding informs decisions:**
  - Data quality issues → cleaning strategy
  - Distributions → preprocessing choices
  - Missingness patterns → imputation decisions
  - Class imbalance → splitting/sampling strategy

### 2. **Pipeline Implementation (25% weight) — Show it works**
- Provide executable evidence (code runs successfully)
- Demonstrate on real data (representative sample or full dataset)
- Show output/screenshots where helpful
- Modular design (separate functions/scripts for each stage)
- Error handling and validation visible in code

### 3. **Repository & Reproducibility (10% weight) — Another user can run it**
- Clear README with exact setup and run steps
- No hardcoded paths or secrets
- Dependencies clearly specified
- Test your instructions in a fresh environment

---

# 4. Submission Checklist

### Before You Submit

- [ ] Report is a single PDF file
- [ ] Repository URL is on first page of report and in Moodle submission text
- [ ] Repository access is verified (team can view it)
- [ ] All required sections are present (see 1.1)
- [ ] EDA is purposeful and interpreted (not automated output)
- [ ] Pipeline diagram corresponds to your actual implementation
- [ ] Pipeline runs without errors on your sample/full dataset
- [ ] README instructions are tested and work in a fresh clone
- [ ] No credentials, API keys, or personal data in repository
- [ ] Commits show meaningful development over time
- [ ] All new evidence from Milestone 1 is highlighted (avoid duplication)

### Submission Process

1. Upload report as single PDF to Moodle (Milestone 2 assignment)
2. Include repository URL in:
   - First page of PDF report
   - Moodle submission text box
3. Verify repository access before deadline
4. Moodle timestamp determines on-time status

---

# 5. Academic Integrity & AI Tools

You may use AI-assisted tools for learning, debugging, editing, and code suggestions where permitted.

**You remain responsible for:**
- Every submitted claim, line of code, citation, and result
- Your submission must demonstrate **your own analysis and understanding**

### Required Actions

- ✅ Acknowledge material use of generative AI (tool, purpose, how output was checked/changed)
- ❌ Do NOT submit fabricated citations, datasets, results, or implementation evidence
- ❌ Do NOT upload confidential, personal, licensed, or restricted data to AI services without permission
- ✅ Be prepared to explain and modify your code during feedback or demonstration

---

# 6. Timeline & Support

**Deadline:** Sunday, 1st November 2026 at 23:59  
**Submission:** Moodle only (not email)

### If Circumstances Affect Your Work
- Contact the teaching team **as early as possible**
- Use the official SETU institutional process for extensions
- ❌ Do NOT rely on Slack messages as formal extension requests

### Feedback
- Identifies strongest evidence and priority corrections
- Outlines work needed before model training (Milestone 3)
- Slack discussion can support formative feedback but does not replace submitted evidence

---

# 7. What Happens Next

After Milestone 2 feedback, you'll be ready to:
- Build and train a baseline model (Milestone 3)
- Use a validated, clean pipeline
- Focus on model selection and evaluation

**Key insight:** The effort you invest in EDA and pipeline quality now directly reduces debugging and rework later.

---

## Quick Reference: Scoring Pathways

### High Marks (70–100%)
- ✅ Deep, interpreted EDA with clear links to decisions
- ✅ Working pipeline with modular, validated code
- ✅ Justified design choices across acquisition, storage, transformation
- ✅ Repository another user can clone and run immediately
- ✅ Clear reflection on limitations and Milestone 3 plan

### Good Marks (60–69%)
- ✅ Sound EDA with reasonable interpretation
- ✅ Core pipeline runs and implements main stages
- ✅ Mostly appropriate design choices with some justification
- ✅ Repository mostly reproducible with minor gaps
- ✅ Main next steps identified

### Developing Marks (40–59%)
- ⚠️ Basic summaries present but incomplete interpretation
- ⚠️ Partially working pipeline with manual steps
- ⚠️ Partial design; some justifications missing
- ⚠️ Repository accessible but requires unstated setup steps
- ⚠️ Limited reflection on next steps

### Insufficient Marks (0–39%)
- ❌ EDA absent or not interpreted
- ❌ Little executable evidence or code failures
- ❌ Generic or unexplained architecture
- ❌ Repository not reproducible or inaccessible
- ❌ No credible plan for Milestone 3

---

**Good luck! Remember: focus on demonstrating your understanding of the data and a working pipeline.**
