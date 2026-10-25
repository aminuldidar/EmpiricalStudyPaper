# Replication Package: The Shadow Supply Chain: An Empirical Study of Method-Level License Violations

This repository contains the replication package, anonymized datasets, and validation data accompanying our empirical study on cross-boundary open-source code reuse and license debt propagation across enterprise software ecosystems.

This package is provided anonymized for double-blind review; the full artifact, including the mining pipeline source code, will be released as a permanent, citable archive on Zenodo once the paper is accepted (the one file already hosted on Zenodo today, `full_compliance_empirical_dataset.csv`, is there purely due to GitHub's file-size limit, not anonymization).

---

## 📂 Repository Structure

```text
.
├── README.md
├── MINER_SETUP.md
├── downsized_compliance_empirical_dataset.csv
├── human_validation.csv
├── compatibility_matrix.csv
├── incompatibility_matrix.csv
├── compatibility_matrix.png
├── incompatibility_matrix.png
├── data_analysis_script.ipynb
├── DSR_engine.py
├── verify_package.ipynb
└── LICENSE
```

**Not included in this git repository, hosted separately on Zenodo (too large for a plain GitHub file — see below):** `full_compliance_empirical_dataset.csv`.

| File                                         | Description                                                                                                                                                            |
| -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `README.md`                                  | Documentation for the replication package.                                                                                                                             |
| `MINER_SETUP.md`                             | Operational model of the mining pipeline that produced these datasets (CLI usage, credential policy, hashing, revisit behavior), shared across companion packages.    |
| `downsized_compliance_empirical_dataset.csv` | Production empirical dataset containing 197,557 method-level reuse records.                                                                                            |
| `human_validation.csv`                       | Dual-annotated, independently re-verified validation dataset containing 1,129 audited method pairs, used for reporting human-agreement and resolution statistics.      |
| `compatibility_matrix.csv` / `.png`          | The top-5 directional license-compatibility matrix reported in the paper, and the figure rendered from it.                                                             |
| `incompatibility_matrix.csv` / `.png`        | The top-5 directional license-incompatibility matrix reported in the paper, and the figure rendered from it.                                                           |
| `full_compliance_empirical_dataset.csv`       | **Hosted on Zenodo: [DOI PLACEHOLDER — fill in after upload].** The full 530,132-record corpus the two matrices above are computed from. Not included here because it exceeds GitHub's 100MB per-file limit without Git LFS; Zenodo supports it directly (up to 50GB per file). |
| `data_analysis_script.ipynb`                 | Jupyter notebook reproducing the empirical analyses, statistical summaries, tables, and evaluation results reported in the paper. Requires `full_compliance_empirical_dataset.csv` (from Zenodo, see above) for the matrix-generating cells.                                      |
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

## 2. `human_validation.csv`

**Description**

The complete dual-annotated validation corpus contains **1,129** method pairs independently reviewed by human experts, plus an independent live re-verification against current repository data. This is the dataset used to evaluate human agreement and automated compliance classification performance.

**Provenance note**: this file is derived from an internal raw annotation file (dual human annotators plus the automated classifier's raw output) retained privately by the authors but not included in this package. During replication-package validation, several of its columns were found to be unreliable or redundant and were dropped rather than shipped; others were renamed for clarity. Both are documented below so the cleaning decisions are auditable without requiring the raw file itself.

**Columns from the raw annotation process not included here, and why**

| Column not included | Reason |
| --- | --- |
| `source_query_project` | Internal pipeline grouping variable; confirmed unreliable as a ground-truth signal. |
| `agreement_flag`, `human_agreement` | Confirmed during validation to track `source_query_project`, not genuine human-vs-automated agreement, despite the naming. |
| `source_license`, `sink_license` | Confirmed to be parsed back out of the `violation` message text rather than independently detected; redundant with `violation`, which is retained. |

**Columns renamed from the raw annotation process**

| Original name | New name |
| --- | --- |
| `cross_val_category` | `automated_classifier_category` |
| `cross_val_message` | `automated_classifier_message` |
| `human_annotator_1`, `human_annotator_2` | `human_rating_1`, `human_rating_2` |

**Columns added**

| Column Name | Type | Description |
| --- | --- | --- |
| `repo_unreachable` | Boolean | True if either repository returned 404 during independent re-verification (repository since deleted, renamed, or moved — "digital decay"). |
| `resolution_category` | Integer (1–5) | The row's resolved category. Populated for all 1,129 rows. |
| `resolution_method` | String | `unanimous_human_agreement` (1,111 rows — the two human raters agreed), `resolved_by_author_review` (12 rows — genuine annotator disagreement, Category 3 vs. 4, resolved by direct re-review), `possible_temporal_drift_unverified` (5 rows — see below), or `possible_detector_limitation_unverified` (1 row — a discrepancy explained by a known GitHub license-detector limitation, not an annotator error). |
| `pending_review_note` | String | Plain-language note explaining why a row isn't a simple unanimous human agreement; does not imply which original rating is correct. |

### Full schema (self-contained, not a diff)

| Column Name | Type                | Description |
| --- |---------------------| --- |
| `organization_name` | String              | Enterprise workspace identifier (e.g., Microsoft, Oracle). |
| `project_type` | Integer             | Internal categorical pipeline tracking identifier. |
| `hash` | String (Hex)        | Cryptographic structural signature used across the extraction pipeline. |
| `source_method_name` | String              | Original method identifier at the source repository. |
| `sink_method_name` | String              | Method identifier at the destination repository. |
| `source_repository_url` | String (URL)        | Fully qualified URL to the upstream repository. |
| `sink_repository_url` | String (URL)        | Fully qualified URL to the downstream repository. |
| `source_file_location` | String              | Source file path and starting line number. |
| `sink_file_location` | String              | Destination file path and starting line number. |
| `violation` | String              | License conflict message (e.g., `GPL-2.0-only` incompatible with `GPL-3.0-only`). |
| `relational_id` | String              | Internal identifier linking extraction sessions. |
| `source_repo_level_license` | String (SPDX)       | Repository-level SPDX license declared by the source project, from an independent live query. |
| `sink_repo_level_license` | String (SPDX)       | Repository-level SPDX license declared by the destination project, from an independent live query. |
| `automated_classifier_category` | Integer (1–4)       | The automated classifier's output (formerly `cross_val_category`). |
| `automated_classifier_message` | String              | Descriptive explanation for that category (formerly `cross_val_message`). |
| `human_rating_1` | Integer (1–4)       | Classification assigned by Expert Annotator 1 (formerly `human_annotator_1`). |
| `human_rating_2` | Integer (1–4)       | Classification assigned by Expert Annotator 2 (formerly `human_annotator_2`). |
| `repo_unreachable` | Boolean             | True if either repository 404'd during independent re-verification. |
| `resolution_category` | Integer (1–5)       | The row's resolved category. |
| `resolution_method` | String              | How the row was resolved — see table above. |
| `pending_review_note` | String              | Plain-language note explaining why a row isn't a simple unanimous human agreement. |

21 columns total. Verified against the live file: exact name and order match, no columns missing or extra.

**On the 5 `possible_temporal_drift_unverified` rows**: a Deutsche Telekom fork relationship with upstream `thingsboard/thingsboard` was recorded as Apache-2.0 on both sides. The live upstream `LICENSE` file is now a Business Source License. An Internet Archive snapshot from 2026-01-05 confirms the file still read Apache License at that date, but no snapshot exists between then and the present, so whether this specific relationship was affected cannot be confirmed from public records alone — reported as a possible, unverifiable instance of drift, not a correction.

**On the 1 `possible_detector_limitation_unverified` row**: GitHub's license detector failed to recognize a real `LICENSE` file due to non-standard header text — a known, confirmed limitation (see the paper's limitations section) — so the recorded value was kept rather than "corrected" to the detector's output.

**On the 12 `resolved_by_author_review` rows**: the two original human raters split 3 vs. 4 on all 12. Each was re-reviewed directly against the current repository: in every case the license is determinable from the file itself but not from the repository's own metadata, which the automated classifier relies on — resolved to Category 5 (Undetermined), not 3 or 4. See `blind_review_sheet.csv` for the per-row detail.

### Validation Results

Reproduced by `verify_package.ipynb`, computed directly from this file:

| Comparison | N | Raw agreement | Cohen's κ |
| --- | --- | --- | --- |
| Human rater 1 vs. human rater 2 (`human_rating_1` vs. `human_rating_2`) | 1,129 | 1,117/1,129 = 98.94% | 0.984 |

All 12 genuine rater disagreements fall at the Category 3/4 boundary (7 rows rated 3 then 4, 5 rows rated 4 then 3).

---

# ⚙️ Illustrative Usage: The Mining Pipeline

The datasets in this package are the output of a method-level mining and indexing pipeline. The pipeline's own source code is not included in this anonymized package, but its operational model (CLI usage, credential policy, hashing, and revisit behavior) is documented in **[`MINER_SETUP.md`](./MINER_SETUP.md)**, shared across this project's companion replication packages so the description stays identical everywhere it appears. The structural signature it produces is carried here as the `hash` column.

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
validation = pd.read_csv("human_validation.csv")

print(validation.head())
print(validation.shape)
print(validation["resolution_method"].value_counts())
```

---

# License

* **Code** (`DSR_engine.py`, `data_analysis_script.ipynb`, `verify_package.ipynb`): released under the **MIT License** (see [`LICENSE`](./LICENSE)).
* **Data** (`downsized_compliance_empirical_dataset.csv`, `human_validation.csv`, `compatibility_matrix.csv`/`.png`, `incompatibility_matrix.csv`/`.png`, `blind_review_sheet.csv`, and the Zenodo-hosted `full_compliance_empirical_dataset.csv`): released under **CC-BY 4.0** (https://creativecommons.org/licenses/by/4.0/).

Repository URLs contained within the datasets refer to their original upstream open-source projects and remain subject to their respective licenses; this package's license applies to our own derived datasets and code, not to the third-party code those URLs point to.
