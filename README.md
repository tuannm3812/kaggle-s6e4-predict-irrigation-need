# Kaggle Playground Series S6E4: Predicting Irrigation Need

This repo is set up for Kaggle Notebooks only. It intentionally does not include local dependency setup or downloaded competition data.

Competition: <https://www.kaggle.com/competitions/playground-series-s6e4>

## Notebooks

- `notebooks/01_eda_predict_irrigation_need.ipynb` - Kaggle-only EDA report for the competition files mounted under `/kaggle/input`.
- `notebooks/02_baseline_models.ipynb` - Kaggle-only baseline modeling notebook with stratified validation, EDA-driven interaction features, CatBoost-first diagnostics, feature interpretation, and submission generation.

## Kaggle Usage

1. Create a new Kaggle Notebook from the competition page.
2. Attach the competition dataset if Kaggle has not attached it automatically.
3. Upload or copy the notebook from this repo into Kaggle.
4. Run all cells.

The notebooks discover `train.csv`, `test.csv`, and `sample_submission.csv` from `/kaggle/input`, infer the target from the sample submission, and avoid local dependency setup.

## Current EDA Summary

Based on the latest Kaggle EDA run:

- Train/test shape: `630,000` training rows, `270,000` test rows.
- Target: 3-class classification with `Irrigation_Need`.
- Class balance: `Low` is about `58.72%`, `Medium` about `37.95%`, and `High` only about `3.33%`.
- Data quality: no missing values, no duplicate rows, and no duplicate IDs.
- Features: `19` predictors, with `11` numeric and `8` categorical.
- Train/test drift is low. The largest numeric standardized mean difference is about `0.004`, and the largest categorical total variation distance is about `0.0025`.
- Strongest individual signals: `Soil_Moisture`, `Rainfall_mm`, `Crop_Growth_Stage`, `Temperature_C`, `Wind_Speed_kmh`, `Previous_Irrigation_mm`, `Humidity`, and `Mulching_Used`.
- Numeric target movement is interpretable: average `Soil_Moisture` drops from `43.31` for `Low` to `17.67` for `High`, while `Temperature_C` rises from `25.35` to `34.57`, and `Wind_Speed_kmh` rises from `9.22` to `14.64`.
- Binned target rates show threshold behavior. The lowest soil-moisture deciles have `High` rates around `9.6%` to `11.7%`, while most higher-moisture deciles are near `0.1%` to `0.2%`. The lowest rainfall decile has a `High` rate around `14.6%`.
- Important categorical patterns: `Crop_Growth_Stage` and `Mulching_Used` look especially predictive. `Flowering` and `Vegetative` have roughly `1.93x` the baseline `High` rate. Fields without mulching have a `High` rate of `5.85%`, compared with `0.79%` for fields with mulching.
- Interaction patterns are strong enough to test in modeling: `Vegetative + No mulching` has about `10.81%` `High`, `Flowering + No mulching` has about `10.61%` `High`, and `Flowering/Vegetative + River` are both around `8.3%` `High`.

## Modeling Plan

The EDA is sufficient to move into experiments. The first modeling notebook starts with:

1. Stratified holdout validation and optional stratified cross-validation.
2. Metrics beyond accuracy: macro F1, weighted F1, balanced accuracy, log loss where available, classification report, and confusion matrices.
3. Conservative baselines: majority-class dummy, one-hot logistic regression, one-hot random forest, histogram gradient boosting, and CatBoost as the primary candidate.
4. EDA-driven interaction features: `Crop_Growth_Stage x Mulching_Used`, `Crop_Growth_Stage x Water_Source`, and `Crop_Growth_Stage x Irrigation_Type`.
5. CatBoost feature-importance interpretation.
6. A final training and `submission.csv` generation section.

Primary modeling risk: the rare `High` class. Accuracy alone will not be enough; each experiment should inspect `High` precision, recall, and confusion with `Medium`.

## Current Baseline Summary

Based on the latest Kaggle baseline run:

- CatBoost is the best holdout model by macro F1: `0.9851` accuracy, `0.9711` macro F1, `0.9638` balanced accuracy, and `0.0602` log loss.
- Histogram gradient boosting is very close on macro F1 and slightly better on log loss, so it remains a useful sanity-check model.
- Random forest has the strongest balanced accuracy among the scikit-learn tree baselines, but its log loss is much weaker.
- CatBoost performs well on the rare `High` class: `0.9653` precision, `0.9205` recall, and `0.9424` F1.
- The first CatBoost submission predicts `Low` at `59.23%`, `Medium` at `37.59%`, and `High` at `3.18%`, close to the training target distribution.

## Next Step

The next notebook should focus on CatBoost tuning and validation, not more broad EDA. Recommended experiments:

1. Run stratified cross-validation for CatBoost to confirm holdout stability.
2. Tune CatBoost depth, learning rate, iterations, L2 regularization, random strength, bagging temperature, and class weights.
3. Compare raw features against the EDA interaction features.
4. Track macro F1, log loss, and `High` precision/recall together.
5. Use feature importance and error analysis to decide whether threshold features from `Soil_Moisture`, `Rainfall_mm`, `Temperature_C`, and `Wind_Speed_kmh` help.
