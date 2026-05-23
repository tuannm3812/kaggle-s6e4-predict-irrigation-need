# 1. Competition Instructions and Solution Approach

## 1.1 Objective

The objective of Kaggle Playground Series S6E4 is to predict the irrigation need
for each test record. The target column is `Irrigation_Need`, a three-class
label with the values `Low`, `Medium`, and `High`.

The submitted file must contain:

- `id`: the test row identifier.
- `Irrigation_Need`: the predicted class label for that row.

The competition rewards correct classification across all irrigation-need
levels. Because the `High` class is rare, model selection should not rely only
on accuracy. Macro F1, balanced accuracy, log loss, and `High`-class recall are
tracked throughout the workflow.

## 1.2 Task Framing

This is a tabular multiclass classification problem. Each row describes crop,
soil, weather, water, and irrigation context. The model must learn patterns that
separate low, medium, and high irrigation need.

Important project constraints:

- Kaggle notebooks are the execution environment.
- Data is loaded from `/kaggle/input`.
- Final reruns should be offline-safe.
- The repository does not store raw competition data.
- Submission notebooks should write `/kaggle/working/submission.csv`.

## 1.3 Key Data Facts

The current analysis records the following dataset facts:

| Item | Value |
| --- | --- |
| Training rows | `630,000` |
| Test rows | `270,000` |
| Target classes | `Low`, `Medium`, `High` |
| Predictors | `19` |
| Numeric predictors | `11` |
| Categorical predictors | `8` |
| Missing values | None observed |
| Duplicate IDs | None observed |

Class balance in train:

| Class | Share |
| --- | ---: |
| `Low` | `58.72%` |
| `Medium` | `37.95%` |
| `High` | `3.33%` |

The rare `High` class makes stratified validation and class-level diagnostics
essential.

## 1.4 Modeling Approach

The project uses a staged notebook workflow:

1. Run exploratory analysis to validate data quality, target balance, feature
   behavior, and train/test alignment.
2. Train transparent baselines to establish a reliable validation frame.
3. Tune CatBoost with compact EDA-driven feature variants.
4. Reuse the selected tuning output in a lightweight submission notebook.

CatBoost is the primary model because it handles mixed numeric and categorical
features naturally and produced the strongest validation results in the
baseline comparison.

## 1.5 Feature Approach

The main feature set keeps the raw competition predictors and adds a small
number of interpretable EDA-driven features:

- `Crop_Growth_Stage` interactions with `Mulching_Used`, `Water_Source`, and
  `Irrigation_Type`.
- Optional threshold indicators for risk-sensitive variables such as
  `Soil_Moisture`, `Rainfall_mm`, and `Temperature_C`.

These features are intentionally compact. The goal is to capture visible
agronomic interactions without creating a large, brittle feature space.

## 1.6 Validation Approach

Validation uses stratification because `High` irrigation need is rare. The
modeling notebooks track:

- accuracy for broad leaderboard intuition;
- macro F1 for equal treatment of all target classes;
- balanced accuracy for class imbalance awareness;
- log loss when probability predictions are available;
- per-class precision, recall, and F1, especially for `High`.

Current best internal result:

| Experiment | Metric |
| --- | ---: |
| CatBoost holdout macro F1 | `0.9711` |
| CatBoost CV `baseline_interactions` macro F1 | `0.96994` |
| CatBoost `High` holdout F1 | `0.9424` |

## 1.7 Submission Strategy

The final submission path is:

1. Generate a tuned CatBoost submission from
   [`3_catboost_tuning.ipynb`](../notebooks/3_catboost_tuning.ipynb).
2. Attach that notebook output as a Kaggle input dataset.
3. Run [`4_reuse_tuning_submission.ipynb`](../notebooks/4_reuse_tuning_submission.ipynb).
4. Validate row count, columns, IDs, missing predictions, and class labels.
5. Write `/kaggle/working/submission.csv`.

Current public leaderboard score:

| Item | Value |
| --- | ---: |
| Public score | `0.96094` |
| Submission rows | `270,000` |

## 1.8 Recommended Next Work

The next useful experiments should be targeted:

1. Build a compact ensemble from the strongest CatBoost and histogram gradient
   boosting variants.
2. Test probability calibration if public/private behavior suggests that
   calibrated probabilities affect decision quality.
3. Analyze `High`-class errors before adding more feature engineering.
4. Keep the reuse notebook as the fast, low-risk submission path.
