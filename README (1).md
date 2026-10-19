# Ensemble learning and Bayesian Optimization for PV power estimation in seven Indonesian cities

Code for the manuscript *"Evaluation of Ensemble Learning and Bayesian Optimization for Photovoltaic Power Prediction Across Seven Indonesian Cities"* (Journal of Renewable Energy and Smart Grid Technology, RAST; revised version).

The study compares Random Forest, XGBoost, LightGBM and AdaBoost, each with default settings and after Bayesian Optimization, and a Ridge stacking ensemble. The task is hourly PV power estimation in Medan, Pontianak, Jakarta, Surabaya, Makassar, Kupang and Jayapura. Linear Regression and a Decision Tree serve as conventional baselines. The conclusions are also checked on measured PV output from the DKASC Alice Springs test facility.

> **Important:** the target `P_kW` in the main dataset is **PVGIS-simulated** output from the PVGIS PV performance model. PVGIS computes it from the same meteorological inputs that are used as features, so it is not measured plant output. Notebook 03 validates the model ranking and the tuning effect on measured data.

## Repository structure

```
notebooks/
  01_pvgis_data_extraction.ipynb              # downloads the PVGIS dataset (7 cities, 2019–2023)
  02_preprocessing_modeling_evaluation.ipynb  # EDA, preprocessing, models, BO, stacking, statistics,
                                              # 5-seed evaluation, ablation, SHAP, error analysis, annual yield
  03_dkasc_measured_validation.ipynb          # validation on measured data (DKASC Alice Springs, Array 33)
data/                                         # empty; input and output files are written here
requirements.txt  LICENSE  CITATION.cff  CHANGELOG.md
```

Notebooks 01 and 02 keep the outputs of the runs reported in the paper. Notebook 03 is provided without outputs; its results are summarised below.

## Where each result comes from

| Paper item | Notebook 02 section |
|---|---|
| Fig. 2 (correlation matrix) | 1 · Pearson correlation |
| Table 1 (search ranges) | 3.2 Bayesian Optimization |
| Table 2 (all models, 5 seeds) | 5.1 Random split → 5.4 `t2` |
| Table 3 (feature-group ablation, XGBoost Tuned) | 5.5 |
| Table 4 (selected hyperparameters) | 4.3 |
| Fig. 3 (residual plots) | 6.1 |
| Ridge weights, prediction correlations | 4.2, 4.4 |
| Paired tests, effect sizes, 1 W threshold | 4.5 |
| Table 5 (BO cost, training and inference time) | 3.2 (BO cost), 4.2 (training and inference) |
| Fig. 4 (gain importance), Fig. 5 (SHAP) | 6.4, 6.5 |
| Table 6 (leave-one-city-out, XGBoost Tuned) | 5.5 |
| Table 7 (time-based split) | 5.5 (XGBoost Tuned), 5.2 (Stacking) |
| Table 8 (per city, 5 seeds) | 5.4 `t8` |
| Irradiance CV vs city MAE | 6.2 |
| Table 9 (annual production) | 6.3 |
| Fig. 6, clipping analysis, monthly analysis | 6.9, 6.7, 6.8 and 6.10 |
| Table 10 (measured DKASC data) | notebook 03 |

The summary tables give MAE and RMSE on the normalised scale and in kW. Multiply the kW values by 1000 to get the watts reported in the paper. For the 1 kWp system, nMAE (%) equals MAE (W) divided by 10.

## Data

### PVGIS (main dataset)
Notebook 01 downloads the dataset from the PVGIS API v5.3 (`seriescalc`, radiation database **PVGIS-ERA5**) with these settings:

| Setting | Value |
|---|---|
| Peak power | 1 kWp |
| PV technology | crystalline silicon (`crystSi`) |
| Mounting | free-standing |
| Tilt / azimuth | 10° / 0° (PVGIS convention, 0° = south) |
| System loss | 14 % |
| Period | 2019–2023, hourly |
| Cities | Medan, Pontianak, Jakarta, Surabaya, Makassar, Kupang, Jayapura |

The output file is `data/pvgis_indonesia_solar_dataset.csv`. It has 306,768 rows, of which 153,518 daytime rows remain after preprocessing.

### DKASC Alice Springs (measured validation data)
DKASC data **are not redistributed here** because the DKA Solar Centre terms of use restrict republication. To reproduce notebook 03:

1. Go to <https://dkasolarcentre.com.au/download> → *DKASC, Alice Springs* → *Single Technologies (full data set with weather data)*.
2. Download **Site 33 — REC, 5.5 kW, poly-Si, fixed, 2012**. The file is named `52-Site_DKA-M16_C-Phase.csv`.
3. Place it in `data/measured/`.

Columns used: `timestamp`, `Active_Power` (kW), `Global_Horizontal_Radiation`, `Diffuse_Horizontal_Radiation` (W/m²) and `Weather_Temperature_Celsius`.
- **Wind speed:** `Wind_Speed` is not used because only 41.8 % of its records are present.
- **Period:** 2014–2016.
- **Filtering:** five-minute data are averaged to hourly values (at least 10 of 12 records per hour). Hours with GHI < 10 W/m² and hours outside 05:00–20:00 are removed, leaving 13,049 hourly records.

Data citation (format requested by the DKA Solar Centre):

> Desert Knowledge Australia Centre. 16/10/2026. Download Data: Array 33 REC. Alice Springs. http://dkasolarcentre.com.au/download, date accessed: 16/10/2026.

Read the [terms and conditions](https://dkasolarcentre.com.au/download/terms-conditions) before using or publishing the data.

## How to run

1. Install the environment with `pip install -r requirements.txt` (Python 3.13), or use Google Colab.
2. In each notebook, set `DATA_DIR` in the **CONFIGURATION** cell at the top. On Colab, the cell mounts Google Drive; change the Drive path to your own folder.
3. Run the notebooks in order: **01 → 02 → 03**. Notebook 03 reads `data/multiseed/ms_random.pkl` from notebook 02 to compare model rankings.

**Runtime and hardware.** The runs used Google Colab: Intel Xeon @ 2.20 GHz, 2 logical cores, 12.7 GB RAM, no GPU.
- **Bayesian Optimization:** about 5,500 s for Random Forest and 4,300 s for AdaBoost.
- **Stacking:** one fit takes about 880 s.
- **Five-seed evaluation:** several hours.
- **Notebook 03:** about 30–60 minutes.

These cells overwrite the saved `.pkl` files, so run notebook 02 cell by cell rather than with *Run all*. Training and inference times vary from run to run; Table 5 reports the values printed in Section 4.2.

## Reproducibility notes

- **Split.** The main split is a 70/15/15 random split with `random_state=42`, and the five-seed evaluation uses seeds 42–46. In the random split, the seed changes both the partition and the model. In the time-based and leave-one-city-out tests, the partitions are fixed and only the model seed changes.
- **Leakage control.** Percentile clipping (1st–99th) and MinMax scaling of features and target are fitted on the training partition only, then applied to validation and test. The target is not clipped.
- **Bayesian Optimization.** Uses the `bayesian-optimization` package with 10 initial points + 50 iterations, objective = −MAE on the validation set (n = 23,028). Integer hyperparameters are truncated with `int()`. Tuning is run once on the seed-42 partition, and the selected values are reused for all seeds.
- **Determinism.** Tuned XGBoost uses `subsample = colsample_bytree = 1.0`, so it is deterministic for a fixed partition. Its standard deviation across seeds is therefore zero in the time-based and leave-one-city-out tests. In LightGBM, `subsample` has no effect because `subsample_freq = 0` (LightGBM default).
- **Supplementary outputs.** Sections 5.2–5.4 also print LightGBM results for the time-based and leave-one-city-out tests. The cell in Section 5.5 reuses their saved files. These LightGBM results are not reported in the paper.
- **Metrics.** MAE and RMSE are given on the normalised scale, in kW, and as a percentage of installed capacity (nMAE/nRMSE).

## Main results (5 seeds, mean ± SD, test set)

| Dataset | Best model | MAE | nMAE | R² |
|---|---|---|---|---|
| PVGIS, 7 cities (1 kWp) | Stacking | 0.739 ± 0.005 W | 0.074 % | 0.999957 |
| DKASC Array 33, measured (5.5 kW) | Stacking | 67.9 ± 1.3 W | 1.23 % | 0.988 |

On the measured data, the model ranking agrees with the PVGIS ranking (Spearman ρ = 0.955, n = 11 configurations). Bayesian Optimization reduced MAE for XGBoost, LightGBM and AdaBoost in all five seeds, and for Random Forest in four of five.

## License and citation

Code is released under the MIT License (see `LICENSE`). Data obtained from PVGIS and DKASC remain subject to their providers' terms. To cite the code, see `CITATION.cff` or the Zenodo record linked in the paper.
