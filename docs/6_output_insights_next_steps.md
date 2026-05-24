# 6. Output Insights and Next Steps

## 6.1 Current State

The current workflow has a strong, stable CatBoost solution:

| Area | Result |
| --- | ---: |
| Best holdout model | CatBoost |
| CatBoost holdout accuracy | `0.98512` |
| CatBoost holdout macro F1 | `0.97109` |
| CatBoost holdout balanced accuracy | `0.96380` |
| CatBoost holdout log loss | `0.06020` |
| Best CV experiment | `baseline_interactions` |
| Best CV macro F1 | `0.96994` |
| Public leaderboard score | `0.96094` |

The train/test prediction mix is also reasonable:

| Class | Train Share | Submission Share |
| --- | ---: | ---: |
| `Low` | `58.72%` | `59.23%` |
| `Medium` | `37.95%` | `37.59%` |
| `High` | `3.33%` | `3.18%` |

The model is not obviously over-predicting or under-predicting one class at the
global level. The main remaining risk is still class-boundary quality,
especially around `High`.

## 6.2 Main Insights

### 6.2.1 CatBoost Is the Right Primary Model

CatBoost produced the best holdout macro F1 and a complete submission path. It
also handles the categorical predictors naturally, which matters because
`Crop_Growth_Stage`, `Mulching_Used`, `Water_Source`, and
`Irrigation_Type` carry real signal.

Histogram gradient boosting is close and has slightly better holdout log loss
(`0.05735` versus CatBoost `0.06020`), so it should remain a useful comparison
model or ensemble member.

### 6.2.2 More CatBoost Complexity Did Not Clearly Help

The tuning notebook tested deeper, regularized, weighted, and threshold-feature
variants. The simple `baseline_interactions` setup still had the best mean CV
macro F1.

| Experiment | Main Observation |
| --- | --- |
| `baseline_interactions` | Best mean CV macro F1 |
| `deeper_interactions` | Better log loss, no macro F1 gain |
| `regularized_interactions` | No meaningful macro F1 gain |
| `threshold_features` | Explicit thresholds did not beat interactions |
| `weighted_interactions` | Higher `High` recall but worse macro F1 and log loss |

This suggests the current model is already close to the useful limit for the
current feature set.

### 6.2.3 `High` Recall Can Improve, but the Trade-Off Is Expensive

The selected model has strong `High` performance:

| Metric | Value |
| --- | ---: |
| `High` precision | `0.9653` |
| `High` recall | `0.9205` |
| `High` F1 | `0.9424` |

Class weighting raised `High` recall to about `0.946`, but macro F1 dropped to
about `0.958` and log loss worsened to roughly `0.074`. That is probably not a
good leaderboard trade unless error analysis shows missing `High` is much more
costly than false `High`.

### 6.2.4 The Strongest Signals Are Agronomically Coherent

The output confirms the EDA story:

- lower `Soil_Moisture` sharply increases irrigation need;
- lower `Rainfall_mm` is associated with higher `High` rates;
- higher `Temperature_C` and `Wind_Speed_kmh` increase risk;
- `Flowering` and `Vegetative` stages carry higher `High` rates;
- no mulching has a much higher `High` rate than mulching.

This reduces concern that the model is exploiting a brittle artifact.

## 6.3 What To Do Next

### Priority 1: Build a Compact Ensemble

Do this next. The best candidate is a small probability ensemble:

1. CatBoost `baseline_interactions`.
2. Histogram gradient boosting from the baseline notebook.
3. Optionally CatBoost `deeper_interactions` because it had better log loss.

Start with simple weighted probability averaging, then choose class labels by
`argmax`. Validate with the same stratified folds and compare macro F1, log
loss, and `High` recall.

Recommended search:

| Ensemble | Weights |
| --- | --- |
| CatBoost only | `1.00` |
| CatBoost + HGB | `0.80 / 0.20` |
| CatBoost + HGB | `0.70 / 0.30` |
| CatBoost + deeper CatBoost | `0.75 / 0.25` |
| CatBoost + HGB + deeper CatBoost | `0.60 / 0.20 / 0.20` |

Stop if CV macro F1 does not improve by at least `0.0003` to `0.0005`.

### Priority 2: Run Error Analysis for `High`

Before adding many more features, inspect where true `High` cases are missed:

- true `High` predicted as `Medium`;
- true `High` predicted as `Low`;
- missed `High` cases by `Crop_Growth_Stage`;
- missed `High` cases by `Mulching_Used`;
- missed `High` cases across soil moisture, rainfall, temperature, and wind
  deciles.

The goal is to find whether missed `High` examples occupy a specific region
that can be targeted with features or thresholds.

### Priority 3: Test Class-Specific Thresholding

Do not jump straight to class weights. Instead, use out-of-fold probabilities
from the selected model and test small decision adjustments:

- predict `High` if `P(High)` exceeds a tuned threshold;
- or predict `High` if `P(High) / P(Medium)` exceeds a tuned ratio;
- keep the default `argmax` unless macro F1 improves.

This may recover some `High` recall with less damage than full class weighting.

### Priority 4: Calibration Check

Because histogram gradient boosting had better log loss, probability calibration
might help. Test calibration only after the ensemble baseline:

- compare raw CatBoost probabilities;
- isotonic calibration on out-of-fold predictions;
- temperature or Platt-style calibration if implementation remains simple.

Select calibration only if it improves log loss without reducing macro F1.

## 6.4 Recommended Next Notebook

Create:

```text
notebooks/5_compact_ensemble_and_thresholds.ipynb
```

The notebook should:

1. reuse the same Kaggle loading and feature functions;
2. train CatBoost and histogram gradient boosting with stratified folds;
3. save out-of-fold probabilities;
4. compare weighted probability ensembles;
5. test conservative `High` threshold adjustments;
6. train the selected final approach on all rows;
7. write `/kaggle/working/submission.csv`.

Keep it compact. The current evidence says careful validation is more valuable
than a wide parameter search.
