# Replication Package: Ecosystem-Scale Method-Level License Compliance Tracking

This repository contains the replication package, anonymized datasets, and validation data accompanying our empirical study on cross-boundary open-source code reuse and license debt propagation across enterprise software ecosystems.

---

## 📂 Repository Structure

```text
.
├── README.md
├── downsized_compliance_empirical_dataset.csv
├── human_oracle_cross_validation.csv
├── data_analysis_script.ipynb
├── DSR_engine.py
└── verify_package.ipynb
```

| File                                         | Description                                                                                                                                                            |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `README.md`                                  | Documentation for the replication package.                                                                                                                             |
| `downsized_compliance_empirical_dataset.csv` | Production empirical dataset containing 197,557 method-level reuse records.                                                                                            |
| `human_oracle_cross_validation.csv`          | Dual-annotated ground-truth validation dataset containing 754 audited method pairs.                                                                                    |
| `data_analysis_script.ipynb`                 | Jupyter notebook reproducing the empirical analyses, statistical summaries, tables, and evaluation results reported in the paper.                                      |
| `DSR_engine.py`                              | Python implementation of the DSR (Detection, Similarity, and Reasoning) engine used to identify license compliance violations and generate compliance classifications. |
| `verify_package.ipynb`                       | Jupyter notebook for verifying the replication package by loading the datasets, reproducing the reported benchmark metrics, and validating the experimental pipeline.  |


# 📊 Dataset Specifications

## 1. `downsized_compliance_empirical_dataset.csv`

**Description**

The production empirical corpus contains **197,557** filtered records representing unique cross-boundary method reuse links mined from global open-source software ecosystems.

**Primary Use**

This dataset is used to study license incompatibilities, licensing drift, and provenance debt propagation across software dependencies.

### Schema

| Column Name             | Type         | Description                                                                             |
| ----------------------- | ------------ | --------------------------------------------------------------------------------------- |
| `organization_name`     | String       | Enterprise workspace identifier (e.g., Accenture).                                      |
| `hash`                  | String (Hex) | Cryptographic canonical AST structural signature of the method subtree (Primary Key).   |
| `source_method_name`    | String       | Original method identifier at the source repository.                                    |
| `sink_method_name`      | String       | Method identifier at the destination repository.                                        |
| `source_repository_url` | String (URL) | Fully qualified URL to the original upstream repository (truncated to ≤100 characters). |
| `sink_repository_url`   | String (URL) | Fully qualified URL to the downstream repository (truncated to ≤100 characters).        |
| `source_file_location`  | String       | Source file path and starting line number.                                              |
| `sink_file_location`    | String       | Destination file path and starting line number.                                         |
| `source_query_project`  | Integer      | Internal pipeline grouping variable.                                                    |
| `violation`             | String       | License compliance conflict identified by the DSR engine.                               |

---

## 2. `human_oracle_cross_validation.csv`

**Description**

The complete dual-annotated validation corpus contains **1,130** method pairs independently reviewed by human experts using a double-blind annotation protocol.

**Primary Use**

This dataset is used to evaluate both human agreement and automated compliance classification performance.

### Schema

| Column Name                 | Type          | Description                                                                       |
| --------------------------- | ------------- | --------------------------------------------------------------------------------- |
| `organization_name`         | String        | Enterprise workspace identifier (e.g., Microsoft, Oracle).                        |
| `project_type`              | Integer       | Internal categorical pipeline tracking identifier.                                |
| `hash`                      | String (Hex)  | Cryptographic structural signature used across the extraction pipeline.           |
| `source_method_name`        | String        | Original method identifier at the source repository.                              |
| `sink_method_name`          | String        | Method identifier at the destination repository.                                  |
| `source_repository_url`     | String (URL)  | Fully qualified URL to the upstream repository.                                   |
| `sink_repository_url`       | String (URL)  | Fully qualified URL to the downstream repository.                                 |
| `source_file_location`      | String        | Source file path and starting line number.                                        |
| `sink_file_location`        | String        | Destination file path and starting line number.                                   |
| `source_query_project`      | Integer       | Internal pipeline grouping variable.                                              |
| `violation`                 | String        | License conflict message (e.g., `GPL-2.0-only` incompatible with `GPL-3.0-only`). |
| `relational_id`             | String        | Internal identifier linking extraction sessions.                                  |
| `source_license`            | String (SPDX) | File-level SPDX license identified for the source artifact.                       |
| `sink_license`              | String (SPDX) | File-level SPDX license identified for the destination artifact.                  |
| `source_repo_level_license` | String (SPDX) | Repository-level SPDX license declared by the source project.                     |
| `sink_repo_level_license`   | String (SPDX) | Repository-level SPDX license declared by the destination project.                |
| `cross_val_category`        | Integer (1–4) | Final validation category assigned during evaluation.                             |
| `cross_val_message`         | String        | Descriptive explanation associated with the assigned validation category.         |
| `agreement_flag`            | String        | Human agreement status (`Agreed` or `Disagreed`).                                 |
| `human_annotator_1`         | Integer (1–4) | Classification assigned by Expert Annotator 1.                                    |
| `human_annotator_2`         | Integer (1–4) | Classification assigned by Expert Annotator 2.                                    |
| `human_agreement`           | String        | Final consensus outcome after reconciliation.                                     |

---

### Confusion Matrix

| Evaluation Population             |   1,130 |


|                   | Predicted Positive (Compliant) | Predicted Negative (Debt) |
| ----------------- | -----------------------------: | ------------------------: |
| **Actual Positive (Compliant)** | 695 | 45 |
| **Actual Negative (Debt)**      | 11  | 379 |

---

# 🛠️ Usage Quickstart

Install the required dependencies:

```bash
pip install pandas scikit-learn
```

Load the production dataset:

```python
import pandas as pd

df = pd.read_csv("downsized_compliance_empirical_dataset.csv")

print(df.head())
print(df.shape)
```

Load the human validation dataset:

```python
validation = pd.read_csv("human_oracle_cross_validation.csv")

print(validation.head())
print(validation.shape)
```
