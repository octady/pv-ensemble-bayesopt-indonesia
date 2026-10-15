# Ensemble learning and Bayesian Optimization for PV power estimation in seven Indonesian cities

Code accompanying the manuscript *"Evaluation of Ensemble Learning and Bayesian Optimization for Photovoltaic Power Prediction Across Multi-Zone Tropical Climates in Indonesia"* (submitted to the Journal of Renewable Energy and Smart Grid Technology, RAST).

The study compares Random Forest, XGBoost, LightGBM and AdaBoost (default and Bayesian-Optimization-tuned) and a Ridge stacking ensemble for hourly PV power estimation in Medan, Pontianak, Jakarta, Surabaya, Makassar, Kupang and Jayapura. It also checks whether the conclusions hold on measured PV output from the DKASC Alice Springs test facility.

> **Important:** the target `P_kW` in the main dataset is **PVGIS-simulated** output from the PVGIS PV performance model. It is computed from the same meteorological inputs used as features. It is not measured plant output. Notebook 03 validates the model ranking and the tuning effect on measured data.

## Repository structure

```
notebooks/
  01_pvgis_data_extraction.ipynb              # downloads the PVGIS dataset (7 cities, 2019–2023)
  02_preprocessing_modeling_evaluation.ipynb  # EDA, preprocessing, models, BO, stacking, 5-seed evaluation,
                                              # ablation, statistics, SHAP, error analysis, annual yield
  03_dkasc_measured_validation.ipynb          # validation on measured data (DKASC Alice Springs, Site 33)
data/                                         # empty; input and output files are written here
requirements.txt
LICENSE
```

Notebook 02 keeps the outputs of the runs reported in the paper. Notebook 03 is provided without outputs; its results are summarised below.

## Data

### PVGIS (main dataset)
Notebook 01 regenerates the dataset from the PVGIS API v5.3 (`seriescalc`, radiation database **PVGIS-ERA5**). It uses these settings:

| Setting | Value |
|---|---|
| Peak power | 1 kWp |
| PV technology | crystalline silicon (`crystSi`) |
| Mounting | free-standing |
| Tilt / azimuth | 10° / 0° (PVGIS convention, 0° = south) |
| System loss | 14 % |
| Period | 2019–2023, hourly |
| Cities | Medan, Pontianak, Jakarta, Surabaya, Makassar, Kupang, Jayapura |

Output: `data/pvgis_indonesia_solar_dataset.csv` (306,768 rows; 153,518 daytime rows after preprocessing).

### DKASC Alice Springs (measured validation data)
DKASC data **are not redistributed here** because the DKA Solar Centre terms of use restrict republication. To reproduce notebook 03:

1. Go to <https://dkasolarcentre.com.au/download> → *DKASC, Alice Springs* → *Single Technologies (full data set with weather data)*.
2. Download **Site 33 — REC, 5.5 kW, poly-Si, fixed, 2012**. The downloaded file is named `52-Site_DKA-M16_C-Phase.csv`.
3. Place it in `data/measured/`.

Columns used: `timestamp`, `Active_Power` (kW), `Global_Horizontal_Radiation`, `Diffuse_Horizontal_Radiation` (W/m²) and `Weather_Temperature_Celsius`. `Wind_Speed` was not used because only 41.8 % of its records are present. Period: 2014–2016. The 5-minute data are averaged to hourly values (at least 10 of 12 records per hour), and hours with GHI < 10 W/m² are removed. This leaves 13,049 hourly records.

Data citation (format requested by the DKA Solar Centre):

> Desert Knowledge Australia Centre. 16/10/2026. Download Data: Array 33 REC. Alice Springs. http://dkasolarcentre.com.au/download, date accessed: 16/10/2026.

Read the [terms and conditions](https://dkasolarcentre.com.au/download/terms-conditions) before using or publishing the data.

## How to run

1. Install the environment: `pip install -r requirements.txt` (Python 3.13). Google Colab also works.
2. Open each notebook and set `DATA_DIR` in the **CONFIGURATION** cell at the top. On Colab the cell mounts Google Drive; change the Drive path to your own folder.
3. Run the notebooks in order: **01 → 02 → 03**. Notebook 03 uses `data/multiseed/ms_random.pkl` from notebook 02 to compare model rankings.

**Runtime.** On the hardware used (Intel Xeon @ 2.20 GHz, 2 cores, 12.7 GB RAM, no GPU), Bayesian Optimization took about 5,500 s for Random Forest and 4,300 s for AdaBoost. One stacking fit took about 880 s, and the 5-seed evaluation takes several hours. These cells overwrite the saved `.pkl` files, so run notebook 02 cell by cell instead of using *Run all*. Notebook 03 takes about 30–60 minutes.

## Reproducibility notes

- **Split.** 70/15/15 random split with `random_state=42`. The multi-seed evaluation uses seeds 42–46. In the random split a seed changes both the split and the model seed; in the time-based and leave-one-city-out tests the partitions are fixed and only the model seed changes.
- **Leakage control.** Percentile clipping (1st–99th) and MinMax scaling of features and target are fitted on the training partition only, then applied to validation and test.
- **Bayesian Optimization.** `bayesian-optimization` package, 10 initial points + 50 iterations, objective = −MAE on the validation set. Integer hyperparameters are truncated with `int()`. The values actually used are written to `data/best_hyperparameters.csv`.
- **LightGBM.** `subsample` has no effect because `subsample_freq=0` (LightGBM default). The tuned LightGBM is therefore deterministic, and its standard deviation across seeds in the time-based and leave-one-city-out tests is zero.
- **Metrics.** MAE/RMSE on the normalised scale, in kW, and as a percentage of installed capacity (nMAE/nRMSE).

## Main results (5 seeds, mean ± SD, test set)

| Dataset | Best model | MAE | nMAE | R² |
|---|---|---|---|---|
| PVGIS, 7 cities (1 kWp) | Stacking | 0.000739 ± 0.000005 kW | 0.074 % | 0.99996 |
| DKASC Site 33, measured (5.5 kWp) | Stacking | 0.0679 ± 0.0013 kW | 1.23 % | 0.988 |

On the measured data, model ranking agrees with the PVGIS ranking (Spearman ρ = 0.955, n = 11 configurations). Bayesian Optimization reduced MAE for XGBoost, LightGBM and AdaBoost in all five seeds, and for Random Forest in four of five.

## License

Code: MIT License (see `LICENSE`). Data obtained from PVGIS and DKASC remain subject to their providers' terms.
