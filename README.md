# Machine-Learning Decision Support for Climate-Resilient Agricultural Water Management

Forecasting next-day soil moisture from climate reanalysis data, and using daily rainfall to size rainwater-harvesting storage for agriculture in the west of Ireland.

**Status:** new research, building on my [MSc dissertation project](https://github.com/srushti28003/ai-stormwater-harvesting). Manuscript in preparation for journal submission.

---

## At a glance

| | |
|---|---|
| **Data** | ~480,000 hourly Copernicus ERA5 and ERA5-Land records → 4,018 daily observations (2015–2025) |
| **Task 1** | One-day-ahead forecast of 0–1 m soil moisture |
| **Models** | Ridge regression, Random Forest, Gradient Boosting, LSTM, against a persistence baseline |
| **Result** | On the unseen 2025 test year, Ridge cut forecast error by **32.8% (RMSE)** and **40.6% (MAE)** versus persistence |
| **Uncertainty** | Moving-block bootstrap (2,000 repetitions): 95% CI for the RMSE reduction **26.3–38.8%** |
| **Task 2** | Daily water-balance simulation of **240** rainwater-harvesting configurations to find the smallest tank meeting 80–95% reliability |
| **Stack** | Python, pandas, NumPy, xarray, scikit-learn, TensorFlow/Keras, SciPy, matplotlib |

## Why it matters

Farms that collect rainwater need to know two things: how wet the soil will be tomorrow, and how big a tank they need to cover demand through dry spells. This project connects a short-term soil-moisture forecast with a transparent storage-sizing screen, so results translate into a practical infrastructure decision rather than a model score alone.

## Method

1. **Data engineering.** Hourly ERA5 weather (wind, temperature, dew point, precipitation) and ERA5-Land soil water (four layers) for a study area near Galway (53.25–53.50° N, 9.00–8.75° W). Unit conversion, relative humidity from dew point, spatial averaging over grid cells, hourly-to-daily aggregation, and a depth-weighted 0–1 m soil-moisture profile. One ERA5-Land cell with no data was excluded rather than imputed.
2. **Leakage-safe features.** Current soil moisture, precipitation, temperature, humidity and wind, plus 3- and 7-day rainfall totals and 1/3/7-day soil-moisture lags, using only information available on day *t*.
3. **Chronological evaluation.** Train 2015–2022, validate 2023–2024, held-out test 2025. Hyperparameters tuned with 5-fold `TimeSeriesSplit` cross-validation inside the training period.
4. **Model selection.** Ridge had the lowest validation error, so the simpler model was chosen, frozen, refitted on 2015–2024 and evaluated once on 2025.
5. **Uncertainty.** Seven-day moving-block bootstrap to respect day-to-day autocorrelation.
6. **Storage screening.** 4 catchment areas × 3 collection coefficients × 4 daily demands × 5 tank sizes, simulated day by day for 2016–2025 with mass-balance and monotonicity checks. Each unmet target is classified as limited by rainfall yield or by storage.

## Results

**Forecasting — 2025 held-out test year**

| Model | MAE (m³/m³) | RMSE (m³/m³) | R² |
|---|---|---|---|
| Persistence baseline | 0.002808 | 0.003707 | 0.9910 |
| **Ridge regression (selected)** | **0.001670** | **0.002492** | **0.9959** |

High R² values partly reflect strong day-to-day persistence in soil moisture, which is why the improvement over the persistence baseline is the main measure of skill.

**Storage screening — example (collection coefficient 0.75, 90% reliability)**

| Catchment | Low demand 0.25 m³/day | Moderate 0.5 | High 1.0 | Very high 2.0 |
|---|---|---|---|---|
| 100 m² | 20 m³ | Yield-limited | Yield-limited | Yield-limited |
| 500 m² | 5 m³ | 5 m³ | 20 m³ | Yield-limited |
| 1,000 m² | 5 m³ | 5 m³ | 10 m³ | 50 m³ |

## Repository contents

| File | Description |
|---|---|
| `soil_moisture_forecasting_and_storage_screening.ipynb` | Full analysis: data preparation, modelling, uncertainty, storage simulation and figures |
| `requirements.txt` | Python packages used |

## Reproducing the analysis

The raw data are not included. Download them from the [Copernicus Climate Data Store](https://cds.climate.copernicus.eu/) for 1 Jan 2015 – 31 Dec 2025 over the study area above:

- **ERA5 hourly data on single levels:** 10 m u and v wind, 2 m temperature, 2 m dew point, surface latent heat flux, total precipitation (CSV), saved as `ERA5_Weather_2015_2025.zip`
- **ERA5-Land hourly data:** volumetric soil water layers 1–4 (NetCDF), saved as `ERA5LAND_SoilMoisture_2015_2025.zip`

The notebook was developed in Google Colab and expects these files in `/content/`. To run locally, change the file paths in the data-loading cells, then:

```bash
pip install -r requirements.txt
jupyter notebook soil_moisture_forecasting_and_storage_screening.ipynb
```

Random seeds are fixed (42) for reproducibility.

## Limitations

- Soil moisture comes from a reanalysis model, not field sensors, so the forecast is evaluated against ERA5-Land rather than in-situ measurements.
- One study area; results are not yet tested in other regions.
- Storage scenarios are hypothetical design cases and do not model evaporation, leakage or first-flush diversion.

## Author

**Srushti Jadhav** · MSc Data Analytics, National College of Ireland
[LinkedIn](https://www.linkedin.com/in/srushti-jadhav60/)

## Data credit

ERA5 and ERA5-Land data © Copernicus Climate Change Service, used under the Copernicus licence.
