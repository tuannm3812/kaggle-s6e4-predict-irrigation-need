# 7. Compact Ensemble and Thresholds

## 7.1 Purpose

[`5_compact_ensemble_and_thresholds.ipynb`](../notebooks/5_compact_ensemble_and_thresholds.ipynb)
tests the next modeling step after CatBoost tuning. It asks whether a small
probability ensemble and conservative `High`-class decision rules can improve
validation quality without starting a broad parameter search.

## 7.2 Logic Flow

| Step | Action | Why It Matters |
| --- | --- | --- |
| 1 | Load Kaggle competition files | Keeps the notebook submission-ready |
| 2 | Rebuild EDA interaction features | Uses the strongest stable feature set |
| 3 | Train base models with stratified folds | Creates comparable OOF probabilities |
| 4 | Evaluate each model by macro F1, log loss, and `High` recall | Keeps class balance central |
| 5 | Blend probabilities with a small weight grid | Tests model diversity without overfitting |
| 6 | Tune conservative `High` rules on OOF probabilities | Checks whether missed `High` cases can be recovered |
| 7 | Select the best CV approach | Avoids leaderboard-only decision making |
| 8 | Train selected models on all rows | Produces final test probabilities |
| 9 | Write `/kaggle/working/submission.csv` | Creates the Kaggle submission artifact |

## 7.3 Base Models

The notebook compares:

| Model | Reason |
| --- | --- |
| `catboost_baseline` | Current strongest tuned solution |
| `catboost_deeper` | Better log loss direction from tuning |
| `hist_gradient_boosting` | Close baseline model with strong probability quality |

The ensemble search intentionally stays small. The current solution is already
strong, so the goal is to capture complementary errors rather than overfit a
large grid.

## 7.4 Ensemble Tests

The first weight grid includes:

| Candidate | Weights |
| --- | --- |
| CatBoost only | `1.00` |
| CatBoost + HGB | `0.80 / 0.20` |
| CatBoost + HGB | `0.70 / 0.30` |
| CatBoost + deeper CatBoost | `0.75 / 0.25` |
| CatBoost + HGB + deeper CatBoost | `0.60 / 0.20 / 0.20` |

Select by macro F1 first, then use log loss and `High` recall as supporting
metrics.

## 7.5 High-Class Rules

The notebook tests two conservative override families:

- predict `High` when `P(High)` exceeds a threshold;
- predict `High` when `P(High) / P(Medium)` exceeds a threshold.

These are safer than broad class weighting because they operate on calibrated
or near-calibrated model confidence rather than changing the whole training
objective.

## 7.6 Decision Rule

Use the new notebook only if it improves cross-validation clearly. A practical
minimum improvement is `0.0003` to `0.0005` macro F1. If the ensemble gain is
smaller, keep the current CatBoost tuning submission as the stable solution.

## 7.7 Current Output Review

The latest run found a small but useful validation gain:

| Candidate | CV Macro F1 | Log Loss | `High` Recall |
| --- | ---: | ---: | ---: |
| CatBoost baseline | `0.96994` | `0.06088` | `0.91294` |
| HGB | `0.96970` | `0.05901` | `0.91651` |
| CatBoost `0.70` + HGB `0.30` | `0.97022` | `0.05934` | `0.91385` |
| Ensemble plus `P(High) >= 0.45` | `0.97032` | `0.05934` | `0.91670` |

The selected candidate uses:

- CatBoost baseline weight `0.70`;
- histogram gradient boosting weight `0.30`;
- `P(High) >= 0.45` as the conservative `High` override.

The generated test prediction mix was:

| Class | Share |
| --- | ---: |
| `Low` | `59.23%` |
| `Medium` | `37.57%` |
| `High` | `3.20%` |

This is still close to the training distribution, so the threshold did not
over-expand the rare class. The next check is leaderboard confirmation.

After the next Kaggle submission, record:

- best individual model metrics;
- best ensemble weights;
- selected `High` rule;
- `High` precision, recall, and F1 before and after the rule;
- submission prediction mix;
- public leaderboard score.
