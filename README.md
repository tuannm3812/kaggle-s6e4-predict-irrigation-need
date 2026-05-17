# Kaggle Playground Series S6E4: Predicting Irrigation Need

This repo is set up for Kaggle Notebooks only. It intentionally does not include local dependency setup or downloaded competition data.

Competition: <https://www.kaggle.com/competitions/playground-series-s6e4>

## Notebooks

- `notebooks/01_eda_predict_irrigation_need.ipynb` - first-pass EDA for the Kaggle competition files mounted under `/kaggle/input`.

## Kaggle Usage

1. Create a new Kaggle Notebook from the competition page.
2. Attach the competition dataset if Kaggle has not attached it automatically.
3. Upload or copy the notebook from this repo into Kaggle.
4. Run all cells.

The EDA notebook discovers `train.csv`, `test.csv`, and `sample_submission.csv` from `/kaggle/input`, infers the target from the sample submission, and then walks through schema checks, missingness, target balance, feature summaries, train/test drift checks, correlations, and quick univariate signal ranking.

