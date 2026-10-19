# Changelog

## v1.1.0 (revised manuscript)

- Notebook 02 now contains the final runs of the revised manuscript. The feature-group ablation (Table 3), the leave-one-city-out test (Table 6) and the time-based test (Table 7) use XGBoost Tuned, the best single model (Section 5.5).
- Removed exploratory cells that are not used in the paper:
  - a single-run Linear Regression and Decision Tree baseline (replaced by the five-seed evaluation);
  - the earlier LightGBM ablation;
  - a LightGBM seed check;
  - a single-seed per-city comparison and its bar chart;
  - a duplicate plotting cell that raised an error.
- Removed Colab-only upload cells. Colab download calls are skipped outside Colab, and all paths are set through `DATA_DIR`.
- Added English section headings and a table in the README that maps every table and figure to a notebook section.
- Added `CITATION.cff`, `.zenodo.json` and this changelog.

## v1.0.0

- First release accompanying the initial submission.
