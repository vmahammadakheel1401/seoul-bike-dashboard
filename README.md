# Seoul Bike Sharing Demand Prediction Dashboard

A small end-to-end project for predicting hourly Seoul bike rental demand from weather, time, and calendar inputs. The repository contains the source dataset, the analysis/training notebook, exported model artifacts, and a static web dashboard that runs predictions entirely in the browser.

**Live demo:** [Seoul Bike Demand Dynamic Predictor](https://vmahammadakheel1401.github.io/seoul-bike-dashboard/)

## What this project does

- Explores and models hourly bike demand using `data/SeoulBikeData.csv` (8,760 hourly records, Dec 2017–Nov 2018).
- Trains and compares regression models in `notebook/seoul_bike_colab.ipynb`.
- Exports a lightweight browser-runnable model to `bike_model.js`.
- Serves a static UI (`index.html`) where users enter conditions and get an estimated bikes/hour prediction immediately on the client side (no backend/API call).

## How the dashboard works

The deployed app is a plain HTML/CSS/JavaScript page:

1. `index.html` loads the transpiled model script: `<script src="bike_model.js"></script>`.
2. The UI collects user inputs:
   - Hour (`0–23`)
   - Temperature (`°C`)
   - Humidity (`%`)
   - Wind speed (`m/s`)
   - Rainfall (`mm`)
   - Snowfall (`cm`)
   - Day type (`Weekday` / `Weekend`)
   - Holiday (`No Holiday` / `Holiday`)
   - Season (`Spring`, `Summer`, `Autumn`, `Winter`)
3. When **Run Model Live** is clicked, `calculateLivePrediction()`:
   - Reads and parses form values.
   - Encodes season as one-hot flags.
   - Builds the feature vector in the exact order expected by the model.
   - Calls the global `score(inputFeatures)` function defined in `bike_model.js`.
   - Clamps negative output to `0`, rounds to the nearest integer, and displays it as `bikes/hour`.
4. All inference happens locally in the browser. If the model script fails to load, the UI shows an alert asking to ensure `bike_model.js` is loaded successfully.

### Model input contract (feature order)

`bike_model.js` exposes:

```js
function score(input) { ... }
```

The input array must follow this exact 12-feature order (as used in `index.html` and the notebook export):

| Index | Feature | Type / encoding | Source in UI |
| --- | --- | --- | --- |
| 0 | `hour` | numeric (`0–23`) | Hour input |
| 1 | `temperature` | numeric (`°C`) | Temperature input |
| 2 | `humidity` | numeric (`%`) | Humidity input |
| 3 | `wind_speed` | numeric (`m/s`) | Wind speed input |
| 4 | `rainfall` | numeric (`mm`) | Rainfall input |
| 5 | `snowfall` | numeric (`cm`) | Snowfall input |
| 6 | `is_weekend` | `0` weekday, `1` weekend | Day type select |
| 7 | `holiday_flag` | `0` no holiday, `1` holiday | Holiday select |
| 8 | `season_Autumn` | one-hot (`0/1`) | Season select |
| 9 | `season_Spring` | one-hot (`0/1`) | Season select |
| 10 | `season_Summer` | one-hot (`0/1`) | Season select |
| 11 | `season_Winter` | one-hot (`0/1`) | Season select |

Example call shape:

```js
score([12, 20.0, 50, 1.5, 0.0, 0.0, 0, 0, 0, 0, 1, 0]);
// [hour, temp, humidity, wind, rain, snow, weekend, holiday, autumn, spring, summer, winter]
```

## Repository structure

```text
seoul-bike-dashboard/
├── index.html                  # Static dashboard UI + client-side prediction logic
├── bike_model.js               # Transpiled browser model exporting global score(input)
├── demand_lookup.json          # Precomputed demand lookup artifact (2,880 rows)
├── data/
│   └── SeoulBikeData.csv       # Source dataset used for analysis/modeling
├── notebook/
│   └── seoul_bike_colab.ipynb  # EDA, training, evaluation, export/generation workflow
└── README.md                   # Project documentation
```

## Data and modeling artifacts

### Dataset (`data/SeoulBikeData.csv`)
The dataset includes hourly rental counts and weather/calendar fields such as:
`Date`, `Rented Bike Count`, `Hour`, `Temperature(°C)`, `Humidity(%)`, `Wind speed (m/s)`, `Visibility (10m)`, `Dew point temperature(°C)`, `Solar Radiation (MJ/m2)`, `Rainfall(mm)`, `Snowfall (cm)`, `Seasons`, `Holiday`, `Functioning Day`.

### Notebook pipeline (`notebook/seoul_bike_colab.ipynb`)
The notebook contains the project workflow, including:
- Data loading/cleaning and exploratory analysis
- Model training and comparison (e.g. Linear Regression, Random Forest, Gradient Boosting, and optional XGBoost where available)
- Definition of the browser feature set (`WEB_FEATURES`) used by the dashboard
- Training/export of a compact web model and generation of browser artifacts

From the notebook:
- **Web feature set:** `['hour', 'temperature', 'humidity', 'wind_speed', 'rainfall', 'snowfall', 'is_weekend', 'holiday_flag'] + season one-hot columns`
- **Web model training (recorded):** `RandomForestRegressor(n_estimators=15, max_depth=8, min_samples_leaf=5, random_state=...)`
- **JS export:** `m2cgen.export_to_javascript(web_model)` written to `bike_model.js`

### Browser model (`bike_model.js`)
- A generated, self-contained JavaScript scoring function (`score`) produced from the compact web Random Forest via `m2cgen`.
- Loaded directly by `index.html` and executed in-browser.
- Large transpiled decision-tree logic (no external runtime dependencies).

### Precomputed lookup table (`demand_lookup.json`)
- Generated in the notebook from the same small web model for a fixed scenario grid.
- Structure: array of objects with keys `season`, `temp`, `rain`, `hour`, `demand` (2,880 rows).
- Grid/assumptions used during generation (from notebook): seasons `Winter/Spring/Summer/Autumn`, temperatures `-10..35` step `5`, rainfall `[0, 1, 5]`, hours `0..23`; with fixed `humidity` (`50` when dry, `85` when rainy), `wind_speed=1.5`, weekday, non-holiday.
- **Note:** the current `index.html` dashboard performs live inference via `bike_model.js` and does not fetch this JSON file.

## Model performance (as recorded in the notebook)

The notebook records a time-based train/test evaluation and a separate compact "web model" export. Reported metrics include:

| Model | R² | RMSE | MAE | Notes |
| --- | ---: | ---: | ---: | --- |
| XGBoost Regressor | 0.803 | 266.20 | 189.75 | Best recorded overall accuracy (not used in browser) |
| Gradient Boosting | 0.706 | 324.71 | 235.99 | Tree ensemble baseline |
| Random Forest | 0.687 | 335.05 | 249.26 | Larger RF variant in notebook comparison |
| Web Random Forest | 0.628 | 365.43 | 258.32 | Compact 12-feature model exported to `bike_model.js` |
| Linear Regression | 0.494 | 426.30 | 320.77 | Linear baseline |

> Metric values are documented from the project notebook/results; rerun the notebook to reproduce or update them for your environment/data split.

## Run locally

### Option 1: Quick open (simple case)
Because the dashboard only loads a local script (`bike_model.js`) and does not fetch external JSON/resources, opening `index.html` directly in a browser may work in many environments.

### Option 2: Serve locally (recommended)
From the repository root:

```bash
python -m http.server 8000
```

Then open: `http://localhost:8000`

Serving over HTTP is a reliable way to avoid browser local-file restrictions and matches common static hosting behavior.

## Reproducing or updating artifacts

Use `notebook/seoul_bike_colab.ipynb` to rerun analysis, retrain models, and regenerate outputs such as:
- `bike_model.js` (browser inference model)
- `demand_lookup.json` (precomputed lookup table)
- additional exported model files referenced in notebook cells

If you change the web feature set or its order during export, update both:
- the model export logic in the notebook, and
- the `inputFeatures` construction/order in `index.html`

## Notes and limitations

- Predictions are estimates based on historical patterns in the provided dataset and the compact exported web model.
- The dashboard does not validate business rules beyond basic numeric parsing/UI constraints; unrealistic combinations may still produce a numeric estimate.
- `demand_lookup.json` is included as a generated artifact but is not currently consumed by the live UI.
- This is a static demo application (no persistence, no backend services, no authentication).
