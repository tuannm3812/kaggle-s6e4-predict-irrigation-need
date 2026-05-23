# 4. CatBoost Tuning

## 4.1 Purpose

[`3_catboost_tuning.ipynb`](../notebooks/3_catboost_tuning.ipynb) takes the
baseline decision and tests whether targeted CatBoost changes improve the
solution. The goal is controlled improvement, not a large blind grid.

The notebook focuses on:

1. feature variants from EDA;
2. compact CatBoost parameter changes;
3. class weighting for the rare `High` class;
4. stratified cross-validation to reduce single-split noise.

## 4.2 Logic Flow

| Step | Action | Why It Matters |
| --- | --- | --- |
| 1 | Load competition files from `/kaggle/input` | Keeps execution identical to Kaggle scoring |
| 2 | Label-encode `Irrigation_Need` | Gives CatBoost numeric class IDs while preserving class names |
| 3 | Build interaction features | Reuses the baseline feature set that already performed well |
| 4 | Build optional threshold features | Tests explicit risk cut points from EDA |
| 5 | Define CatBoost experiment dictionaries | Keeps feature frames, columns, and parameters tied together |
| 6 | Run stratified CV for each experiment | Gives a more stable estimate than one holdout split |
| 7 | Store out-of-fold predictions and probabilities | Enables confusion matrix and class-level diagnostics |
| 8 | Rank experiments by macro F1 | Optimizes for all classes, not only the majority class |
| 9 | Inspect log loss and `High` recall | Checks probability quality and rare-class behavior |
| 10 | Train selected model on full data and write `submission.csv` | Produces the tuned Kaggle artifact |

## 4.3 Experiment Set

| Experiment | Feature Set | Main Change |
| --- | --- | --- |
| `baseline_interactions` | EDA interaction features | Strong baseline CatBoost configuration |
| `deeper_interactions` | EDA interaction features | More depth and iterations, lower learning rate |
| `regularized_interactions` | EDA interaction features | Stronger regularization and bagging temperature |
| `weighted_interactions` | EDA interaction features | Class weights to help rare `High` recall |
| `threshold_features` | Interactions plus threshold indicators | Adds explicit soil/weather risk flags |

The threshold indicators are intentionally small. They test whether CatBoost
benefits from explicit high-risk boundaries for variables such as
`Soil_Moisture`, `Rainfall_mm`, `Temperature_C`, and `Wind_Speed_kmh`.

## 4.4 Metrics

The tuning notebook uses the same metric philosophy as the baseline notebook:

- macro F1 is the primary selection metric;
- log loss is tracked for probability quality;
- balanced accuracy checks class-imbalance robustness;
- per-class recall highlights whether `High` improves or degrades.

Class-weighted experiments are judged carefully. A higher `High` recall is only
useful if it does not damage macro F1 or probability quality too much.

## 4.5 Results

| Experiment | Result |
| --- | ---: |
| Best CV macro F1 | `baseline_interactions` at `0.96994` |
| Mean log loss for best macro F1 model | `0.06088` |
| Best log loss direction | `deeper_interactions` around `0.06016` |
| Class-weighted `High` recall | around `0.946` |
| Selected submission strategy | `baseline_interactions` |

The deeper model improved log loss but did not improve macro F1. The weighted
model raised `High` recall, but that came with lower macro F1 and worse log
loss. The selected solution therefore favors the best class-balanced validation
result instead of chasing a single class metric.

## 4.6 Interpretation

The tuning results suggest that the baseline feature interactions already
capture most of the available signal. More complexity does not clearly improve
the competition objective.

The practical conclusion is:

- CatBoost is already near the useful performance range for this feature set.
- `High` recall can be pushed upward with class weights, but the trade-off is
  not free.
- Additional work should be evidence-driven: error analysis, calibration, or a
  compact ensemble rather than another broad CatBoost search.

## 4.7 Submission Output

The selected model is trained on the full training set and writes:

```text
/kaggle/working/submission.csv
```

That output is reused by
[`4_reuse_tuning_submission.ipynb`](../notebooks/4_reuse_tuning_submission.ipynb)
to avoid rerunning the long tuning notebook just to resubmit.
