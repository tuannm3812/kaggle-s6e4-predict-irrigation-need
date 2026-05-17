# Kaggle Playground Series S6E4: Predicting Irrigation Need

This repo is set up for Kaggle Notebooks only. It intentionally does not include local dependency setup or downloaded competition data.

Competition: <https://www.kaggle.com/competitions/playground-series-s6e4>

## Notebooks

- `notebooks/01_eda_predict_irrigation_need.ipynb` - Kaggle-only EDA for the competition files mounted under `/kaggle/input`.

## Kaggle Usage

1. Create a new Kaggle Notebook from the competition page.
2. Attach the competition dataset if Kaggle has not attached it automatically.
3. Upload or copy the notebook from this repo into Kaggle.
4. Run all cells.

The EDA notebook discovers `train.csv`, `test.csv`, and `sample_submission.csv` from `/kaggle/input`, infers the target from the sample submission, and then walks through schema checks, missingness, target balance, feature summaries, train/test drift checks, correlations, univariate signal ranking, categorical lift, interaction slices, and deeper train/test drift diagnostics.

## Current EDA Summary

Based on the first Kaggle run:

- Train/test shape: `630,000` training rows, `270,000` test rows.
- Target: 3-class classification with `Irrigation_Need`.
- Class balance: `Low` is about `58.72%`, `Medium` about `37.95%`, and `High` only about `3.33%`.
- Data quality: no missing values, no duplicate rows, and no duplicate IDs.
- Features: `19` predictors, with `11` numeric and `8` categorical.
- Train/test drift: numeric feature means are extremely close; the largest standardized mean difference is only about `0.004`.
- Strong first-pass signals: `Soil_Moisture`, `Rainfall_mm`, `Crop_Growth_Stage`, `Temperature_C`, `Wind_Speed_kmh`, `Previous_Irrigation_mm`, `Humidity`, and `Mulching_Used`.
- Important categorical patterns: `Crop_Growth_Stage` and `Mulching_Used` look especially predictive. `Flowering` and `Vegetative` are much more likely to be `Medium` or `High`, while `Sowing` and `Harvest` skew heavily toward `Low`. Fields without mulching have a much higher `High` rate than fields with mulching.

## Recommended Next Steps

The dataset looks clean and stable, so the next notebook should focus on modeling rather than heavy data cleaning:

1. Use stratified cross-validation because the `High` class is rare.
2. Track class-level metrics, not just accuracy.
3. Build a simple baseline with categorical handling, likely starting with tree-based models available in Kaggle.
4. Compare raw categorical features against targeted interaction features such as growth stage by mulching, irrigation type, and water source.
5. Calibrate or tune class behavior if the model under-predicts `High`.
