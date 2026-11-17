# 🚲 Seoul Bike Sharing Demand Prediction & Web Dashboard

An end-to-end machine learning project that forecasts hourly bike rental demand in Seoul based on weather conditions, seasonal trends, and calendar data. This repository includes both the predictive modeling pipeline and a lightweight, client-side interactive web dashboard deployed via GitHub Pages.

🔗 **Live Demo:** [Seoul Bike Demand Dynamic Predictor](https://vmahammadakheel1401.github.io/seoul-bike-dashboard/)

---

## 📌 Project Overview

Urban bike-sharing systems rely heavily on accurate demand forecasting to manage station rebalancing, fleet maintenance, and staff allocation. Using historical hourly rental and meteorological data (`SeoulBikeData.csv`, 8,760 hourly records), this project explores demand patterns, benchmarks multiple regression algorithms, and compiles a lightweight Random Forest model directly into JavaScript (`bike_model.js`) for instant, serverless browser predictions.

### Key Insights Discovered
* **Commuter & Leisure Peaks:** Hourly demand shows strong bimodal peaks during morning and evening rush hours on weekdays, shifting to afternoon peaks on weekends and holidays.
* **Weather Sensitivity:** Temperature is the strongest positive driver of demand up to ~25°C, whereas extreme heat (35°C) drops daily demand by **~42%**, heavy rain (5 mm/h) drops demand by **~65%**, and winter snow reduces demand by over **82%**.
* **System Operations:** Non-functioning system days (295 hours) were filtered out during preprocessing to prevent zero-inflated bias.

---

## 🛠️ Tech Stack & Architecture

* **Data Analysis & Modeling:** Python, Pandas, NumPy, Matplotlib, Seaborn, Scikit-Learn, XGBoost
* **Model Compilation & Export:** `m2cgen` (transpiles Scikit-Learn Random Forest into standalone JavaScript), `joblib`
* **Generative AI Integration:** Google GenAI SDK (`gemini-2.5-flash`) for automated operational briefings
* **Frontend & Deployment:** HTML5, CSS3, Vanilla JavaScript, GitHub Pages

---

## 📊 Model Performance

Models were evaluated using a realistic **time-based 80/20 split** (training on Dec 2017 – Sep 2018, testing on Sep 2018 – Nov 2018) to prevent data leakage and simulate real-world forecasting:

| Model | $R^2$ Score | RMSE | MAE | Notes |
| :--- | :---: | :---: | :---: | :--- |
| **XGBoost Regressor** | **0.803** | **266.20** | **189.75** | Best overall accuracy (saved as `.joblib`) |
| **Gradient Boosting** | 0.706 | 324.71 | 235.99 | Strong baseline tree ensemble |
| **Random Forest** | 0.687 | 335.05 | 249.26 | Full 16-feature model |
| **Web Random Forest** | 0.628 | 365.43 | 258.32 | Pruned 12-feature model exported to `bike_model.js` |
| **Linear Regression** | 0.494 | 426.30 | 320.77 | Linear baseline |

---

## 📂 Repository Structure

```text
seoul-bike-dashboard/
├── index.html          # Interactive web UI for live demand prediction
├── bike_model.js       # Transpiled Random Forest model running client-side in JS
└── README.md           # Project documentation
