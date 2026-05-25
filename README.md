# Kaggle Playground S6E4: Predicting Irrigation Need

![Irrigation banner](https://media.hswstatic.com/eyJidWNrZXQiOiJjb250ZW50Lmhzd3N0YXRpYy5jb20iLCJrZXkiOiJnaWZcL0lycmlnYXRpb24uanBnIiwiZWRpdHMiOnsicmVzaXplIjp7IndpZHRoIjoiMTIwMCJ9fX0=)

![Kaggle](https://img.shields.io/badge/Kaggle-Playground%20S6E4-20BEFF?style=flat-square&logo=kaggle&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![CatBoost](https://img.shields.io/badge/Model-CatBoost-FFCC00?style=flat-square)
![Public Score](https://img.shields.io/badge/Public%20Score-0.96094-success?style=flat-square)
![Workflow](https://img.shields.io/badge/Workflow-Kaggle%20Notebook-lightgrey?style=flat-square)

This project solves Kaggle Playground Series S6E4 as a notebook-first tabular
classification workflow. The task is to predict `Irrigation_Need` as `Low`,
`Medium`, or `High` from crop, soil, weather, water-source, and irrigation
context.

The repository is intentionally Kaggle-native: notebooks load data from
`/kaggle/input`, write submissions to `/kaggle/working/submission.csv`, and avoid
committing raw competition data.

## Project Snapshot

| Item | Result |
| --- | ---: |
| Training rows | `630,000` |
| Test rows | `270,000` |
| Predictors | `19` raw features |
| Target classes | `Low`, `Medium`, `High` |
| Best public score so far | `0.96094` |
| Best stable CV model | CatBoost `baseline_interactions` |
| Best ensemble CV test | CatBoost + HGB with `P(High) >= 0.45` |

## Technical Skills Demonstrated

| Area | Evidence in Project |
| --- | --- |
| Exploratory data analysis | Data quality checks, target balance, drift checks, feature-target relationships |
| Tabular ML | CatBoost, histogram gradient boosting, logistic regression, random forest baselines |
| Imbalanced classification | Stratified validation, macro F1, balanced accuracy, `High`-class precision/recall tracking |
| Feature engineering | Growth-stage interactions and EDA-driven threshold tests |
| Model validation | Holdout, stratified cross-validation, OOF probabilities, confusion matrices, log loss |
| Ensembling | Weighted probability blending across CatBoost and HGB models |
| Decision optimization | Conservative `High` probability threshold and `High`/`Medium` ratio tests |
| Kaggle workflow | Offline-safe notebooks, `/kaggle/input` discovery, validated `submission.csv` generation |
| Documentation | Notebook companion docs, output insights, coding standards, reproducible run notes |

## Repository Structure

```text
.
├── README.md
├── docs/
│   ├── 1_instructions.md
│   ├── 2_eda_insights.md
│   ├── 3_baseline_models.md
│   ├── 4_catboost_tuning.md
│   ├── 5_reuse_tuning_submission.md
│   ├── 6_output_insights_next_steps.md
│   ├── 7_compact_ensemble_and_thresholds.md
│   └── coding_standards.md
└── notebooks/
    ├── 1_eda_predict_irrigation_need.ipynb
    ├── 2_baseline_models.ipynb
    ├── 3_catboost_tuning.ipynb
    ├── 4_reuse_tuning_submission.ipynb
    └── 5_compact_ensemble_and_thresholds.ipynb
```

No `data/`, `models/`, or `outputs/` folders are required for the committed
workflow. Kaggle-mounted inputs and generated submissions stay outside git.

## Run Instructions

### Option A: Full Kaggle Workflow

1. Open the Kaggle competition notebook environment for Playground Series S6E4.
2. Attach the competition dataset if it is not attached automatically.
3. Upload or copy notebooks from `notebooks/`.
4. Run notebooks in order:
   - `1_eda_predict_irrigation_need.ipynb`
   - `2_baseline_models.ipynb`
   - `3_catboost_tuning.ipynb`
   - `5_compact_ensemble_and_thresholds.ipynb`
5. Submit the generated `/kaggle/working/submission.csv`.

### Option B: Fast Reuse Submission

Use this when re-submitting the saved CatBoost tuning output without retraining.

1. Attach the output from the tuning notebook:
   <https://www.kaggle.com/code/tuannm3823/s6e4-predicting-irrigation-need-catboost-tuning?scriptVersionId=320060317>
2. Run `notebooks/4_reuse_tuning_submission.ipynb`.
3. Confirm validation passes: row count, columns, ID order, non-missing labels,
   and allowed target classes.
4. Submit `/kaggle/working/submission.csv`.

## Notebook Workflow

| Notebook | Purpose | Main Output |
| --- | --- | --- |
| `notebooks/1_eda_predict_irrigation_need.ipynb` | Understand data quality, drift, target balance, and feature signal | EDA findings and modeling implications |
| `notebooks/2_baseline_models.ipynb` | Compare model families with stratified validation | CatBoost selected as strongest baseline |
| `notebooks/3_catboost_tuning.ipynb` | Test compact CatBoost variants and feature sets | `baseline_interactions` selected by CV macro F1 |
| `notebooks/4_reuse_tuning_submission.ipynb` | Validate and reuse saved tuning output | Fast, low-risk `submission.csv` |
| `notebooks/5_compact_ensemble_and_thresholds.ipynb` | Test CatBoost/HGB probability blends and `High` rules | Best CV test: `0.97032` macro F1 |

## Documentation Map

| Document | Purpose |
| --- | --- |
| `docs/coding_standards.md` | Notebook-first standards for this Kaggle project |
| `docs/1_instructions.md` | Competition objective, task framing, and solution approach |
| `docs/2_eda_insights.md` | Detailed EDA findings and modeling implications |
| `docs/3_baseline_models.md` | Logic flow, approach, and results for baseline modeling |
| `docs/4_catboost_tuning.md` | Logic flow, approach, and results for CatBoost tuning |
| `docs/5_reuse_tuning_submission.md` | Validation flow and results for submission reuse |
| `docs/6_output_insights_next_steps.md` | Output interpretation and recommended next experiments |
| `docs/7_compact_ensemble_and_thresholds.md` | Logic flow, results, and decision rules for the ensemble notebook |

## Output Review

### Data and EDA

The dataset is clean and stable enough for standard cross-validation:

- no missing values, duplicate rows, or duplicate IDs were observed;
- train/test drift is low, with largest numeric standardized mean difference
  around `0.004`;
- the target is imbalanced: `Low 58.72%`, `Medium 37.95%`, `High 3.33%`.

The strongest signals are agronomically coherent: lower soil moisture, lower
rainfall, higher temperature, higher wind speed, high-risk growth stages, and no
mulching all increase irrigation-need risk.

### Modeling Results

| Model / Experiment | Key Result |
| --- | --- |
| CatBoost holdout | Accuracy `0.9851`, macro F1 `0.9711`, log loss `0.0602` |
| CatBoost `High` class | Precision `0.9653`, recall `0.9205`, F1 `0.9424` |
| CatBoost CV `baseline_interactions` | Mean macro F1 `0.96994`, mean log loss `0.06088` |
| HGB CV | Mean macro F1 `0.96970`, mean log loss `0.05901` |
| CatBoost + HGB ensemble | Mean macro F1 `0.97022`, log loss `0.05934` |
| Ensemble + `P(High) >= 0.45` | Mean macro F1 `0.97032`, `High` recall `0.91670` |

The ensemble improves CV macro F1 by about `0.00038` over the CatBoost CV
baseline and materially improves log loss. The gain is small but real enough to
submit and compare, while the original CatBoost submission remains the stable
fallback.

### Submission State

The validated CatBoost tuning submission has:

| Item | Value |
| --- | ---: |
| Public score | `0.96094` |
| Submission rows | `270,000` |
| Prediction mix: `Low` | `59.23%` |
| Prediction mix: `Medium` | `37.59%` |
| Prediction mix: `High` | `3.18%` |

The ensemble notebook generated a new candidate prediction mix:

| Class | Share |
| --- | ---: |
| `Low` | `59.23%` |
| `Medium` | `37.57%` |
| `High` | `3.20%` |

## Current Lessons

1. CatBoost is the strongest single-model path, but HGB adds useful probability
   diversity.
2. More CatBoost depth improves log loss slightly but does not beat the simpler
   interaction model on macro F1.
3. Full class weighting is too blunt: it raises `High` recall but damages macro
   F1 and log loss.
4. A conservative `High` probability threshold is a better trade-off than class
   weighting because it nudges only borderline cases.
5. The feature story is stable and interpretable, so the next gains should come
   from validation, ensembling, and decision rules rather than large feature
   churn.

## Next Work

1. Submit the ensemble candidate from `5_compact_ensemble_and_thresholds.ipynb`
   and compare it against the current `0.96094` public score.
2. If the ensemble improves the leaderboard, update `4_reuse_tuning_submission`
   or add a reuse notebook for the ensemble output.
3. Run targeted error analysis for true `High` rows predicted as `Medium` or
   `Low`.
4. Test probability calibration only if it improves log loss without reducing
   macro F1.
5. Keep the current CatBoost tuning submission as the stable fallback if the
   ensemble does not generalize.
