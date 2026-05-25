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
| Best ensemble CV test | CatBoost + HGB with `P(High) >= 0.45` |
| Best ensemble CV macro F1 | `0.97032` |
| Public leaderboard score | `0.96094` |

The train/test prediction mix is also reasonable:

| Class | Train Share | Submission Share |
| --- | ---: | ---: |
| `Low` | `58.72%` | `59.23%` |
| `Medium` | `37.95%` | `37.59%` |
| `High` | `3.33%` | `3.18%` |

The model is not obviously over-predicting or under-predicting one class at the
global level. The main remaining risk is class-boundary quality, especially
around `High`.

The latest ensemble notebook produced a new candidate submission mix:

| Class | Ensemble Candidate Share |
| --- | ---: |
| `Low` | `59.23%` |
| `Medium` | `37.57%` |
| `High` | `3.20%` |

## 6.2 Main Insights

### 6.2.1 CatBoost Is the Right Primary Model

CatBoost produced the best holdout macro F1 and a complete submission path. It
also handles the categorical predictors naturally, which matters because
`Crop_Growth_Stage`, `Mulching_Used`, `Water_Source`, and
`Irrigation_Type` carry real signal.

Histogram gradient boosting is close and has better probability quality:
its CV log loss was `0.05901`, compared with CatBoost baseline `0.06088`.
That made it useful as a probability-ensemble member even though its macro F1
was slightly lower.

### 6.2.2 The Compact Ensemble Is the Best Current Experiment

The tuning notebook showed that more CatBoost complexity did not clearly help.
The ensemble notebook then showed that model diversity helped more than another
CatBoost-only variant.

| Experiment | Main Observation |
| --- | --- |
| `baseline_interactions` | Best mean CV macro F1 |
| `deeper_interactions` | Better log loss, no macro F1 gain |
| `regularized_interactions` | No meaningful macro F1 gain |
| `threshold_features` | Explicit thresholds did not beat interactions |
| `weighted_interactions` | Higher `High` recall but worse macro F1 and log loss |
| CatBoost `0.70` + HGB `0.30` | Improved CV macro F1 to `0.97022` |
| CatBoost `0.70` + HGB `0.30` plus `P(High) >= 0.45` | Best CV macro F1 at `0.97032` |

The ensemble gain is modest, about `0.00038` macro F1 over the CatBoost CV
baseline, but it is the best validation-backed improvement so far.

### 6.2.3 `High` Recall Can Improve, but the Trade-Off Is Expensive

The selected model has strong `High` performance:

| Metric | Value |
| --- | ---: |
| `High` precision | `0.9653` |
| `High` recall | `0.9205` |
| `High` F1 | `0.9424` |

Class weighting raised `High` recall to about `0.946`, but macro F1 dropped to
about `0.958` and log loss worsened to roughly `0.074`. The ensemble's
`P(High) >= 0.45` rule is a cleaner trade-off: it raised `High` recall to
`0.91670` while keeping macro F1 at `0.97032`.

### 6.2.4 The Strongest Signals Are Agronomically Coherent

The output confirms the EDA story:

- lower `Soil_Moisture` sharply increases irrigation need;
- lower `Rainfall_mm` is associated with higher `High` rates;
- higher `Temperature_C` and `Wind_Speed_kmh` increase risk;
- `Flowering` and `Vegetative` stages carry higher `High` rates;
- no mulching has a much higher `High` rate than mulching.

This reduces concern that the model is exploiting a brittle artifact.

## 6.3 What To Do Next

### Priority 1: Submit the Ensemble Candidate

Do this next. The best current candidate is:

1. CatBoost `baseline_interactions` at weight `0.70`.
2. Histogram gradient boosting at weight `0.30`.
3. A conservative `P(High) >= 0.45` override.

This candidate improved macro F1 and log loss in cross-validation. Submit it
once, compare with the current `0.96094` public score, and keep CatBoost as the
fallback if the leaderboard does not confirm the CV gain.

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

### Priority 3: Refine Class-Specific Thresholding

The first threshold pass found `P(High) >= 0.45` as the best CV rule. A follow-up
can test a narrower grid:

- `P(High)` thresholds from `0.42` to `0.48`;
- `P(High) / P(Medium)` thresholds around `0.75` to `0.85`;
- no rule if macro F1 or public score gets worse.

Keep this conservative. Large `High` expansion already proved costly under
class weighting.

### Priority 4: Calibration Check

Because histogram gradient boosting had better log loss, probability calibration
might help. Test calibration only after the ensemble baseline:

- compare raw CatBoost probabilities;
- isotonic calibration on out-of-fold predictions;
- temperature or Platt-style calibration if implementation remains simple.

Select calibration only if it improves log loss without reducing macro F1.

## 6.4 Recommended Next Action

Run and submit:

```text
notebooks/5_compact_ensemble_and_thresholds.ipynb
```

If the public leaderboard improves, promote the ensemble path as the active
submission workflow. If it does not, keep the CatBoost tuning submission as the
stable solution and use the ensemble results as a documented experiment.
