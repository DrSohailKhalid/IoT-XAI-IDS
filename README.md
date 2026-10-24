# Explainable AI for IoT Intrusion Detection

Reference implementation with ResDNN, CNN and LSTM classifiers, SHAP and LIME
explanations, and genetic algorithm (GA) and particle swarm optimization (PSO)
searches over explainer settings.

**Authors:** Sohail Khalid, Mujeeb Ur Rehman, Muhammad Abrar, Fahad Algarni,
and Insaf Ullah.

**Scope:** The supplied configurations classify selected attack categories.
Benign/normal traffic is excluded. These experiments do not measure whether a
system can distinguish normal traffic from attacks.

This software accompanies the study titled *Enhancing IoT Security with
Metaheuristic-Optimized Explainable AI Models for Intrusion Detection*, as
identified in the supplied project metadata. The manuscript and a verified
publication DOI were not included in this package. No published performance
claim or exact reproduction of paper results is asserted here.

## Start here

| Task | Instructions |
|---|---|
| Install and run locally | [Local execution](#local-execution) |
| Run in Google Colab | [Colab instructions](docs/COLAB_INSTRUCTIONS.md) |
| Run in Kaggle | [Kaggle instructions](docs/KAGGLE_INSTRUCTIONS.md) |
| Obtain compatible CSV files | [Datasets](docs/DATASETS.md) |
| Understand the implementation | [Methods-to-code map](docs/METHODS_TO_CODE_MAP.md) |
| Interpret results and limitations | [Scope and limitations](docs/SCOPE_AND_LIMITATIONS.md) |
| See what was actually checked | [Validation record](docs/VALIDATION.md) |
| Upload this project to your account | [GitHub upload guide](docs/GITHUB_UPLOAD_GUIDE.md) |

## What the workflow does

1. Load CSV files and map the selected attack labels.
2. Partition rows into training, validation, and test subsets.
3. Fit categorical encoding, KNN imputation, Min-Max scaling, and correlation
   screening on the training subset.
4. Apply SMOTE to the training subset only.
5. Train ResDNN, 1D-CNN, 2D-CNN, and LSTM models and evaluate held-out rows.
6. Explain the configured classifier with SHAP and LIME.
7. Optionally tune SHAP settings with GA and LIME settings with PSO.

The classifier stays fixed during explainer optimization. GA and PSO do not
optimize classifier weights or classification accuracy.

## Datasets and class mapping

Datasets are obtained separately and are not included in this repository.

| Evaluation | Data sources | Retained classes |
|---|---|---|
| `concat` | CICIDS2017 and UNSW-NB15 | DDoS, DoS, Reconnaissance, Bot |
| `cse` | CSE-CIC-IDS2018 | DDoS, DoS, BruteForce, Infiltration |

UNSW-NB15 contributes DoS and Reconnaissance rows to `concat`; the DDoS and Bot
rows come from CICIDS2017. The concatenated data are pooled and randomly split.
This is not a train-on-one-dataset, test-on-another evaluation.

Official sources:

- [CICIDS2017, Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2017.html)
- [UNSW-NB15, UNSW](https://research.unsw.edu.au/projects/unsw-nb15-dataset)
- [CSE-CIC-IDS2018, Canadian Institute for Cybersecurity](https://www.unb.ca/cic/datasets/ids-2018.html)

Use headered flow-feature CSVs with the expected multiclass labels. See
[dataset preparation](docs/DATASETS.md) before mixing releases or mirrors.

## Local execution

Use an isolated Python environment. The lightweight checks are configured for
Python 3.11 and 3.12; that configuration does not establish full TensorFlow/GPU
compatibility on both versions or on every operating system. See the
[validation record](docs/VALIDATION.md) for the environment actually checked.

Run commands from the repository root, where `README.md` and `requirements.txt`
are located.

```bash
python -m venv .venv
```

Activate it on Windows PowerShell:

```powershell
.\.venv\Scripts\Activate.ps1
```

Or on macOS/Linux:

```bash
source .venv/bin/activate
```

Install dependencies:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Place compatible files in `data/cicids2017/`, `data/unsw_nb15/`, and
`data/cse_cic_ids2018/`. Paths can also point to other locations.

Start with a reduced run of the combined dataset:

```bash
python -m src.cli --config configs/quick_check.yaml --benchmark concat --cicids2017 "data/cicids2017/*.csv" --unsw "data/unsw_nb15/*.csv"
```

Or a reduced CSE run:

```bash
python -m src.cli --config configs/quick_check.yaml --benchmark cse --cse "data/cse_cic_ids2018/*.csv"
```

For the larger workflow, replace `configs/quick_check.yaml` with
`configs/full_experiment.yaml`. The full configuration uses 50 training epochs;
optional cross-validation and conventional ML baselines are disabled by default.

The quick configuration retains up to 500 mapped rows per class and source in
source-file order. This is a functionality check, not a representative sample
for performance reporting. It can still scan large files and runs XAI searches
by default. Set `xai.enabled: false` in a copy of the configuration for a
classifier-only check. Re-enabling it starts the pipeline again; there is no
standalone resume-XAI command.

## Outputs

Each run creates a fresh directory such as `outputs/concat_run/` or
`outputs/concat_run_02/`.

| Output | Contents |
|---|---|
| `config_resolved.yaml` | Settings used for that run |
| `data_summary.json` | Retained row counts, feature count, classes |
| `selected_features.json` | Selected and removed feature names |
| `*.keras` | Saved neural models |
| `*_history.csv`, `*_accuracy.png` | Training histories and plots |
| `all_model_test_metrics.json` | Test metrics for the neural models |
| `*_per_class_metrics.csv`, `*_confusion_matrix.csv` | Class-specific metrics and confusion matrices |
| `model_selection.json` | Validation-loss model selection and XAI model name |
| `xai_optimization_summary.json` | Explanation scores, timings, selected settings |
| `ga_shap_history.csv`, `pso_lime_history.csv` | Optional search histories |

Additional files depend on enabled stages. Generated data and outputs are
excluded from Git tracking. No datasets, pretrained weights, benchmark results,
or recorded full-dataset runs are shipped in this repository.

For a publication run, retain the exact source-file list and checksums, the
resolved configuration, package versions, hardware details, and the complete
output directory. Dependency ranges are not an exact environment lock.

## Tests

Run the lightweight tests without installing TensorFlow, SHAP, or LIME:

```bash
python -m pip install -r requirements-test.txt
python -m pytest -q
```

The supplied three lightweight tests cover label mapping and a synthetic
preprocessing workflow. An additional TensorFlow test checks that the 2D-CNN
can be saved and reloaded with matching predictions; it is skipped when
TensorFlow is absent. They do not validate benchmark accuracy, all dataset releases, or the
entire training and explanation pipeline. The GitHub Actions workflow runs
these checks on pushes and pull requests after upload; its remote status is
not claimed before it executes.

For a small end-to-end check without downloading datasets, install the full
dependencies and run:

```bash
python scripts/smoke_check.py
```

This uses artificial data only. Its metrics must not be reported as benchmark
results. Outputs are written under `outputs/synthetic_smoke/`.

## Repository layout

| Location | Purpose |
|---|---|
| `src/` | Loading, preprocessing, models, evaluation, XAI and optimization |
| `configs/` | Quick and full experiment settings |
| `notebooks/` | Colab and Kaggle entry points |
| `tests/` | Lightweight tests and an optional TensorFlow serialization test |
| `scripts/` | Synthetic workflow check |
| `docs/` | Data, execution, validation and upload guidance |
| `data/`, `outputs/` | Local input/output directories, contents ignored |
| `.github/workflows/` | Automatic lightweight checks |

## Citation and license

[CITATION.cff](CITATION.cff) contains the supplied software version and author
metadata. The metadata version is 1.0.0; it does not establish that a GitHub
release or a Zenodo record already exists. Add verified paper and repository
URLs after they are available. No DOI is invented or reserved by this package.

The source code is distributed under the supplied [MIT License](LICENSE).
Dataset access, attribution, and redistribution remain governed by each
provider's terms. See [CONTRIBUTING.md](CONTRIBUTING.md) for contributions and
[CHANGELOG.md](CHANGELOG.md) for preparation changes.
