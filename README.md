# Kaggle Playground Series S6E4: Predicting Irrigation Need

![Irrigation banner](https://media.hswstatic.com/eyJidWNrZXQiOiJjb250ZW50Lmhzd3N0YXRpYy5jb20iLCJrZXkiOiJnaWZcL0lycmlnYXRpb24uanBnIiwiZWRpdHMiOnsicmVzaXplIjp7IndpZHRoIjoiMTIwMCJ9fX0=)

![Kaggle](https://img.shields.io/badge/Kaggle-Playground%20S6E4-20BEFF?style=flat-square&logo=kaggle&logoColor=white)
![Python](https://img.shields.io/badge/Python-3.x-3776AB?style=flat-square&logo=python&logoColor=white)
![CatBoost](https://img.shields.io/badge/Model-CatBoost-FFCC00?style=flat-square)
![Public Score](https://img.shields.io/badge/Public%20Score-0.96094-success?style=flat-square)
![Workflow](https://img.shields.io/badge/Workflow-Kaggle%20Notebooks%20Only-lightgrey?style=flat-square)

This repository contains a Kaggle-only workflow for the [Playground Series S6E4 competition](https://www.kaggle.com/competitions/playground-series-s6e4). It intentionally avoids local dependency setup and downloaded data; all notebooks are designed to run with Kaggle-mounted inputs under `/kaggle/input`.

## Progress Snapshot

| Area | Status | Key Result |
| --- | --- | --- |
| EDA | Complete | Clean dataset, low train/test drift, strong signal from soil moisture, growth stage, mulching, temperature, and wind speed |
| Baseline modeling | Complete | CatBoost selected as primary model with holdout macro F1 `0.9711` |
| Tuning | Complete | `baseline_interactions` remained best by CV macro F1 `0.96994` |
| Submission reuse | Complete | Validated tuning output and submitted public score `0.96094` |
| Next work | Planned | Compact ensemble or calibration experiment |

## Notebook Workflow

| Notebook | Purpose | Notes |
| --- | --- | --- |
| `notebooks/01_eda_predict_irrigation_need.ipynb` | Exploratory data analysis | Data quality, target balance, feature behavior, drift checks, and EDA summary |
| `notebooks/02_baseline_models.ipynb` | Baseline modeling | Dummy, logistic regression, random forest, histogram gradient boosting, CatBoost, diagnostics, and feature importance |
| `notebooks/03_catboost_tuning.ipynb` | CatBoost tuning | Stratified CV, feature variants, class weighting, threshold features, and tuned submission generation |
| `notebooks/04_reuse_tuning_submission.ipynb` | Fast submission reuse | Validates and reuses the long tuning run output without retraining |

## Kaggle Usage

1. Open the competition on Kaggle and create a new notebook.
2. Attach the competition dataset if Kaggle has not attached it automatically.
3. Upload or copy the target notebook from this repo.
4. Run all cells.

For final submission reuse, attach the output from:
<https://www.kaggle.com/code/tuannm3823/s6e4-predicting-irrigation-need-catboost-tuning?scriptVersionId=320060317>

Then run `notebooks/04_reuse_tuning_submission.ipynb`.

## Key EDA Findings

- Train/test shape: `630,000` training rows and `270,000` test rows.
- Target: 3-class `Irrigation_Need` classification.
- Class balance: `Low 58.72%`, `Medium 37.95%`, `High 3.33%`.
- Data quality: no missing values, no duplicate rows, and no duplicate IDs.
- Feature set: `19` predictors, with `11` numeric and `8` categorical.
- Train/test drift is low: largest numeric standardized mean difference is about `0.004`; largest categorical total variation distance is about `0.0025`.
- Strongest signals: `Soil_Moisture`, `Rainfall_mm`, `Crop_Growth_Stage`, `Temperature_C`, `Wind_Speed_kmh`, `Previous_Irrigation_mm`, `Humidity`, and `Mulching_Used`.

The most important risk pattern is consistent: lower soil moisture, lower rainfall, higher temperature, higher wind speed, high-risk growth stages, and no mulching are associated with greater irrigation need.

## Modeling Results

CatBoost is the strongest baseline model and remains the best practical submission path.

| Model / Experiment | Validation Result |
| --- | --- |
| CatBoost holdout | Accuracy `0.9851`, macro F1 `0.9711`, balanced accuracy `0.9638`, log loss `0.0602` |
| CatBoost `High` class | Precision `0.9653`, recall `0.9205`, F1 `0.9424` |
| CatBoost CV `baseline_interactions` | Mean macro F1 `0.96994`, mean log loss `0.06088` |
| CatBoost CV `deeper_interactions` | Better log loss `0.06016`, but no macro F1 improvement |
| Class-weighted CatBoost | Higher `High` recall around `0.946`, but lower macro F1 and worse log loss |

Feature importance confirms the EDA story. The top CatBoost drivers are:

1. `Soil_Moisture`
2. `Crop_Growth_Stage`
3. `Mulching_Used`
4. `Temperature_C`
5. `Wind_Speed_kmh`

EDA interaction features help, but they are secondary to the core agronomic variables.

## Submission Result

The reused CatBoost tuning submission was validated and submitted.

| Item | Value |
| --- | --- |
| Public score | `0.96094` |
| Best score | `0.96094` |
| Submission rows | `270,000` |
| Required columns | `id`, `Irrigation_Need` |
| Prediction mix | `Low 59.23%`, `Medium 37.59%`, `High 3.18%` |

The public score is slightly below the internal CV macro F1 estimate (`0.96994`). This is expected because CV and the public leaderboard are evaluated on different rows, but future improvements should be validated carefully rather than judged from a single holdout split.

## Next Steps

The broad EDA, baseline comparison, and CatBoost tuning work are complete. The next useful modeling direction is not another large CatBoost grid.

Recommended next experiments:

1. Build a compact ensemble using the strongest CatBoost and histogram gradient boosting variants.
2. Test probability calibration if leaderboard behavior suggests calibration matters.
3. Analyze errors for the `High` class before adding more features.
4. Keep `04_reuse_tuning_submission.ipynb` as the fast path for validating and resubmitting saved outputs.
