# Coding Standards

## 1. Repository Scope

This repository is intentionally notebook-first. Kaggle notebooks are the
executable source of truth, while `docs/` captures analysis, model results, and
project decisions for the Playground Series S6E4 irrigation-need competition.

Keep the root small:

- `notebooks/` for Kaggle notebooks.
- `docs/` for instructions, EDA notes, result summaries, and notebook companion
  documentation.
- `README.md` for the high-level project overview.

Avoid adding local-only folders such as `data/`, `models/`, `outputs/`,
`configs/`, or `scripts/` unless the project direction changes back to local
training. Competition data and Kaggle outputs should stay outside git.

## 2. Notebook Naming

Use numbered, stable notebook names that match the actual competition workflow:

1. `1_eda_predict_irrigation_need.ipynb`
2. `2_baseline_models.ipynb`
3. `3_catboost_tuning.ipynb`
4. `4_reuse_tuning_submission.ipynb`
5. `5_compact_ensemble_and_thresholds.ipynb`

Notebook names should describe the Kaggle action performed in the file. Do not
split training and submission into separate notebooks when the intended Kaggle
flow is end-to-end. A dedicated reuse notebook is acceptable when it validates
and republishes a long training run's saved `submission.csv`.

## 3. Code Style

Follow PEP 8 for Python code:

- Use 4 spaces for indentation.
- Keep lines to 79 characters or fewer where practical; allow slightly longer
  lines in notebook display calls when breaking them would hurt readability.
- Prefer concise syntax such as list comprehensions, f-strings, and small
  utility functions when they make the notebook easier to scan.
- Add type hints for reusable functions when the type is clear.
- Group imports in this order:
  1. Standard library
  2. Third-party libraries
  3. Local modules, if any are ever introduced
- Separate import groups with a blank line.

Use Google-style docstrings for reusable functions:

```python
def add_interaction_features(df: pd.DataFrame) -> pd.DataFrame:
    """Add EDA-driven categorical interaction features.

    Args:
        df: Input feature frame.

    Returns:
        Copy of the input frame with additional interaction columns.
    """
```

Add short inline comments only when they explain why a decision was made. Avoid
comments that restate what the code already says.

## 4. Notebook Style

Each notebook should include:

- a short purpose statement at the top;
- a clear configuration section near the top for tunable values;
- explicit mode flags when runtime behavior differs between training,
  validation, and submission;
- Kaggle path auto-detection where practical;
- Markdown insight cells after important plots or metrics;
- artifact-writing cells for reusable outputs such as `submission.csv`,
  histories, or plots.

Prefer readable, self-contained notebook code over imports from local project
modules. Kaggle should be able to run each notebook after attaching only the
required competition datasets and previous notebook outputs.

When notebook code changes, clear all outputs before committing and rerun the
notebook on Kaggle to regenerate trusted outputs. Keep committed notebooks
lightweight; Kaggle is the execution record.

Competition notebooks should not depend on internet access during final reruns.
For runtime packages that differ from the Kaggle image, attach a Kaggle input
dataset containing wheels or model artifacts and load them explicitly. If an
exploratory install is allowed, gate it behind an explicit config flag and keep
the default offline-safe.

Submission notebooks must be optimized for Kaggle scoring limits. Do not run
EDA, avoid unnecessary model training in scored submission paths, and keep
submission mode focused on loading inputs, inference, validation, and writing
`submission.csv`.

## 5. Plot Style

Use the Viridis palette as the default visual language across notebooks:

- Use `sns.color_palette("viridis", ...)` for categorical or sequential
  accents.
- Use `"viridis"` as the default colormap for heatmaps and distribution plots.
- Change color palettes only when a chart needs clearer contrast, semantic
  coloring, or accessibility improvement.
- Keep chart titles short and analytical; avoid decorative styling.

For this project, charts should emphasize irrigation risk patterns: soil
moisture, rainfall, growth stage, temperature, wind speed, previous irrigation,
humidity, and mulching.

## 6. Documentation Style

Documentation should be written for a competition reviewer or teammate who
wants the reasoning quickly:

- Use numbered sections.
- Lead with findings and implications.
- Include exact metrics when available.
- Link notebooks and docs with relative paths.
- Keep detailed notebook notes in separate companion markdown files.
- Keep broad narrative in the root `README.md`; keep detailed evidence in
  focused docs.

The main documentation set is:

- `docs/1_instructions.md` for competition objectives, task framing, and the
  selected solution approach.
- `docs/2_eda_insights.md` for dataset and feature insights.
- `docs/3_baseline_models.md` and later numbered docs for detailed
  notebook-by-notebook modeling explanations.

## 7. Git Hygiene

Do not commit:

- raw Kaggle competition data;
- local checkpoints;
- Kaggle working directories;
- large cached feature tables;
- Python caches or notebook checkpoints;
- ad hoc experiment dumps;
- generated `submission.csv` files unless there is a specific reason to
  preserve one as a lightweight reference.

Commit lightweight artifacts only when they directly support the written
analysis, such as small figures used by EDA markdown and model result pages.
