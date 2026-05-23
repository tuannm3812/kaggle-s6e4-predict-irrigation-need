# 5. Reuse Tuning Submission

## 5.1 Purpose

[`4_reuse_tuning_submission.ipynb`](../notebooks/4_reuse_tuning_submission.ipynb)
is the fast, low-risk submission notebook. It does not train a model. Instead,
it validates the `submission.csv` produced by the long CatBoost tuning run and
copies it into `/kaggle/working/submission.csv`.

## 5.2 Logic Flow

| Step | Action | Why It Matters |
| --- | --- | --- |
| 1 | Attach the CatBoost tuning notebook output as a Kaggle input | Makes the saved tuned submission available without retraining |
| 2 | Find the competition `sample_submission.csv` | Defines required row count, column names, and ID order |
| 3 | Find the tuning `submission.csv` | Selects the model output that should be submitted |
| 4 | Exclude competition-mounted files when searching for the tuning output | Avoids accidentally selecting the sample submission |
| 5 | Compare row count and columns | Prevents malformed submissions |
| 6 | Compare ID order exactly | Prevents row alignment errors |
| 7 | Check missing and unexpected labels | Prevents invalid target predictions |
| 8 | Write `/kaggle/working/submission.csv` | Produces Kaggle's expected output artifact |

## 5.3 Required Input

Attach the output dataset from the CatBoost tuning notebook:

<https://www.kaggle.com/code/tuannm3823/s6e4-predicting-irrigation-need-catboost-tuning?scriptVersionId=320060317>

The notebook prefers paths containing the tuning notebook slug:

```text
s6e4-predicting-irrigation-need-catboost-tuning
```

## 5.4 Validation Rules

The notebook fails early if any of these checks fail:

- tuning submission row count differs from the sample submission;
- column names or order differ;
- IDs are not in the exact same order;
- any prediction is missing;
- predictions contain labels outside `Low`, `Medium`, and `High`.

These checks are simple, but they protect against the highest-cost submission
mistakes.

## 5.5 Results

| Item | Value |
| --- | ---: |
| Public leaderboard score | `0.96094` |
| Submission rows | `270,000` |
| Required columns | `id`, `Irrigation_Need` |
| Prediction mix: `Low` | `59.23%` |
| Prediction mix: `Medium` | `37.59%` |
| Prediction mix: `High` | `3.18%` |

The prediction mix is close to the training target distribution while slightly
under the training `High` share. That is consistent with the tuning results:
the selected model avoids over-expanding the rare class even though weighted
experiments could increase `High` recall.

## 5.6 Decision

Keep this notebook minimal. Its job is validation and reuse, not modeling. If a
new tuned or ensembled submission becomes the selected solution, attach that new
output dataset and reuse the same validation flow.
