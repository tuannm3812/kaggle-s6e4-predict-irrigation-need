# 3. Baseline Models

## 3.1 Purpose

[`2_baseline_models.ipynb`](../notebooks/2_baseline_models.ipynb) turns the EDA
findings into the first reliable modeling benchmark. The notebook answers three
questions:

1. Which model family is strongest before heavy tuning?
2. Do compact EDA interaction features improve validation quality?
3. Which model should become the primary submission path?

The approach is deliberately practical: use a stratified holdout split first,
compare several model families, inspect class-level behavior, then train the
selected model on all training rows for a baseline `submission.csv`.

## 3.2 Logic Flow

| Step | Action | Why It Matters |
| --- | --- | --- |
| 1 | Load `train.csv`, `test.csv`, and `sample_submission.csv` from `/kaggle/input` | Keeps the notebook Kaggle-native and submission-column safe |
| 2 | Infer `id_col` and `target_col` from `sample_submission.csv` | Avoids hard-coding submission schema |
| 3 | Split raw predictors into numeric and categorical groups | Allows each baseline model to use appropriate preprocessing |
| 4 | Add EDA-driven interaction features | Tests whether growth-stage context improves irrigation-risk separation |
| 5 | Encode target labels with `LabelEncoder` | Gives all scikit-learn and CatBoost models a common target representation |
| 6 | Build a stratified holdout split | Preserves the rare `High` class in validation |
| 7 | Fit each baseline model | Compares simple, linear, tree, gradient boosting, and CatBoost approaches |
| 8 | Score accuracy, macro F1, balanced accuracy, and log loss | Balances leaderboard intuition with class-imbalance awareness |
| 9 | Inspect confusion matrix and feature importance for the best model | Checks whether the selected model fails on `High` or uses plausible features |
| 10 | Fit the selected model on full training data and write `submission.csv` | Produces a complete Kaggle baseline submission |

## 3.3 Feature Approach

The notebook starts from all competition columns except `id` and
`Irrigation_Need`. It then adds three categorical interactions from EDA:

| Feature | Reason |
| --- | --- |
| `Crop_Growth_Stage__x__Mulching_Used` | Captures stage-specific water retention benefit |
| `Crop_Growth_Stage__x__Water_Source` | Captures how water source may matter differently by growth stage |
| `Crop_Growth_Stage__x__Irrigation_Type` | Captures stage-specific irrigation delivery behavior |

This keeps the feature set compact and avoids creating a large synthetic feature
space before a baseline is established.

## 3.4 Model Families

| Model | Preprocessing | Role in the Baseline |
| --- | --- | --- |
| Dummy classifier | None | Sanity-check floor |
| Logistic regression | Median imputation, one-hot encoding, scaling | Linear benchmark with class balancing |
| Random forest | Median/mode imputation and ordinal encoding | Nonlinear tree benchmark |
| Histogram gradient boosting | Median/mode imputation and ordinal encoding | Strong scikit-learn gradient boosting baseline |
| CatBoost | Native categorical feature indices | Primary mixed-type tabular candidate |

CatBoost is included because the problem has meaningful categorical variables
and interactions. It can consume categorical columns directly instead of relying
on one-hot or ordinal approximations.

## 3.5 Evaluation Design

The target is imbalanced, so accuracy is not enough. The notebook tracks:

- accuracy for broad correctness;
- macro F1 for equal treatment of `Low`, `Medium`, and `High`;
- balanced accuracy for class-imbalance sensitivity;
- log loss when model probabilities are available;
- per-class precision, recall, and F1 from the classification report.

`High` recall and `High` F1 receive extra attention because `High` irrigation
need is the rare class and the easiest class to under-predict.

## 3.6 Results

CatBoost became the strongest baseline and the best practical model family for
the next notebook.

| Result | Value |
| --- | ---: |
| Best baseline model | CatBoost |
| Holdout accuracy | `0.9851` |
| Holdout macro F1 | `0.9711` |
| Holdout balanced accuracy | `0.9638` |
| Holdout log loss | `0.0602` |
| `High` precision | `0.9653` |
| `High` recall | `0.9205` |
| `High` F1 | `0.9424` |

The `High` class result is strong but still the main risk area. The model is
precise when it predicts `High`, but recall shows that some true `High` cases
are still pulled into lower-risk classes.

## 3.7 Interpretation

CatBoost feature importance matches the EDA story. The top drivers are:

1. `Soil_Moisture`
2. `Crop_Growth_Stage`
3. `Mulching_Used`
4. `Temperature_C`
5. `Wind_Speed_kmh`

This is a useful sanity check: the best baseline is not winning through obscure
leakage-like artifacts. It relies on agronomically plausible water-balance and
growth-stage signals.

## 3.8 Decision

The baseline notebook chooses CatBoost as the primary solution path because it:

- gives the best validation balance;
- handles numeric and categorical columns naturally;
- benefits from small EDA interaction features;
- produces a complete submission workflow without local scripts.

The next step is focused CatBoost tuning, not a broad model search. Histogram
gradient boosting remains a candidate for a later compact ensemble.
