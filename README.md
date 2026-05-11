# Energy Optimization & Electricity Forecasting — Melting Furnace (v6.0)

A full-stack industrial energy intelligence system for a melting furnace. The system combines real-time anomaly detection, prescriptive analysis, multi-granularity historical analytics, and a 15-day energy consumption forecast, all served through an interactive web dashboard.

---

## Background

The dataset was synthetically generated to simulate one year of realistic furnace operation, incorporating real-world defects such as power spikes, power drops, power factor deviations, and idle-time anomalies. After understanding the underlying process parameters, the data was used to train two ML models: one for forecasting and one for anomaly detection.

The entire pipeline — from raw telemetry to actionable insight, was iterated across multiple versions, with v6.0 being the final production-ready implementation.

---

## What the System Does

| Capability | Description |
|---|---|
| Real-Time Monitoring | Streams minute-wise power and power factor readings with live KPI cards |
| Anomaly Detection | XGBoost classifier flags 5 anomaly types per minute and tracks running counts |
| Prescriptive Analysis | Each anomaly maps to a possible reason, root cause, and recommended action |
| Energy Forecasting | Prophet model generates a 15-day consumption forecast with confidence intervals |
| Historical Analysis | Monthly, weekly, daily, and batch-level consumption trends |
| Model Validation | Actual vs predicted overlay for the 11-day test window |

---

## Project Structure

```
6.0/
├── app.py                                        # Flask backend — API routes + model serving
├── Forecast_model.py                             # Prophet forecasting model — training script
├── Energy_anomaly_detection_model.py             # XGBoost anomaly classifier — training script
│
├── forecast_model.pkl                            # Saved Prophet model
├── xgb_anomaly_model.pkl                         # Saved XGBoost classifier
├── label_encoder.pkl                             # Label encoder for anomaly classes
│
├── 1_minuteWiseData_Melting_energy_3.0_DB2.csv   # Minute-level telemetry (17 MB, ~1 year)
├── 2_Melting_batch_data.csv                      # Batch-level aggregated data
├── 2_Melting_Day_level_data.csv                  # Daily aggregated data for forecasting
├── test results.csv                              # Forecast model test predictions
│
├── templates/
│   └── dashboard.html                            # Reactive dark-themed web dashboard
│
└── Model outputs/                                # Saved plots and CSVs from model runs
    ├── energy_efficiency_outputs/
    ├── forecast_comparison_outputs/
    ├── forecast_outputs/
    └── xgboost_retrain_outputs/
```

---

## Dataset

| Property | Value |
|---|---|
| Period | January 1 – May 11, 2025 |
| Granularity | Minute-wise, Batch-wise, Daily |
| Minute Records | ~188,000+ rows |
| Batches | ~3,144 total |
| Columns | timestamp, batch_id, melting_batch_time, idle_batch_time, power_kw, power_factor, furnace_temperature, batch_status, energy_reading_pm, energy_reading_cumm + anomaly flags |

The dataset was synthetically generated to reflect realistic industrial furnace behaviour, then augmented with injected defects to represent:
- Voltage fluctuations (power drops below 350 kW)
- Overload events (power spikes above 400 kW)
- Reactive power inefficiency (power factor < 0.70)
- Over-compensation (power factor > 0.95)

---

## ML Models

### Anomaly Detection — XGBoost Classifier

**File:** [Energy_anomaly_detection_model.py](Energy_anomaly_detection_model.py)

Trained on minute-level features to classify each data point into one of five classes in real time.

| Anomaly Class | Condition | Meaning |
|---|---|---|
| `normal` | Power 350–400 kW, PF 0.70–0.95 | No issue |
| `power_drop` | Power < 350 kW during batch | Possible voltage fluctuation |
| `power_spike` | Power > 400 kW during batch | Possible overload |
| `low_pf` | Power factor < 0.70 | Inefficient reactive power usage |
| `high_pf` | Power factor > 0.95 | Over-compensation |

**Features:** melting_batch_time, idle_batch_time, power_kw, power_factor, furnace_temperature, batch_status, energy_reading_pm, energy_reading_cumm

**Outputs:** `xgb_anomaly_model.pkl`, `label_encoder.pkl`

---

### Energy Forecasting — Prophet

**File:** [Forecast_model.py](Forecast_model.py)

Trained on daily aggregated data to forecast total energy consumption for the next 15 days, with uncertainty intervals.

**Exogenous Regressors:** production (kg), anomaly_intensity, batch_count, is_weekend, is_month_end

**Train/Test Split:**
- Train: Jan 1 – Apr 30, 2025
- Test: May 1–11, 2025
- Forecast: 15 days ahead

**Metrics Reported:** MAE, RMSE, R²

**Outputs:** `forecast_model.pkl`, `future_forecast.csv`, `combined_results.csv`, `forecast_plot.png`

---

## Dashboard

**File:** [templates/dashboard.html](templates/dashboard.html)  
**Backend:** [app.py](app.py)

Dark-themed, fully responsive single-page dashboard built with Chart.js and Plotly.

### Live Monitoring Panel
- Current Power (kW) with normal band overlay (350–400 kW)
- Current Power Factor with normal band overlay (0.70–0.95)
- KPI cards: Power, Power Factor, Today's Consumption, Anomaly Count, System Status

### Anomaly & Prescription Panel
- Latest anomaly badge with type label
- Running counters per anomaly class
- Prescription box showing: Possible Reason / Root Cause / Recommended Solution

### Historical Analysis Panel
- Monthly consumption trend
- Weekly consumption trend
- Daily consumption with anomaly intensity (dual-axis)
- Yesterday's batch-wise consumption vs anomaly count (dual-axis)

### Model Results Panel
- Actual vs predicted consumption for the 11-day test window

### Forecast Panel
- Configurable forecast: date range, production target (kg), anomaly intensity
- 15-day forecast chart with confidence bands + anomaly intensity overlay

### Auto-Refresh
- Real-time data: every 10 seconds
- Historical data: every 5 minutes
- Error handling: 3-strike failure detection before alert

---

## API Endpoints

| Route | Method | Description |
|---|---|---|
| `/` | GET | Serves the dashboard |
| `/data` | GET | Streams real-time minute-wise anomaly data |
| `/historical_data` | GET | Monthly, weekly, daily trends |
| `/batch_data` | GET | Batch-wise consumption and anomaly counts |
| `/test_results` | GET | Actual vs predicted model results |
| `/forecast` | POST | Generates a 15-day forecast on demand |

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Python, Flask |
| ML — Classification | XGBoost, scikit-learn |
| ML — Forecasting | Prophet (Meta) |
| Data Processing | pandas |
| Model Persistence | joblib |
| Frontend Charts | Plotly.js, Chart.js |
| Frontend UI | HTML5, CSS3 (dark theme) |

---

## How to Run

**1. Install dependencies**
```bash
pip install flask xgboost prophet scikit-learn pandas joblib
```

**2. (Optional) Retrain models**
```bash
python Energy_anomaly_detection_model.py
python Forecast_model.py
```

**3. Start the dashboard**
```bash
python app.py
```

Open `http://localhost:5000` in your browser.

---

## Key Design Decisions

- **Synthetic data with injected defects** — allowed full control over anomaly distribution and edge case coverage without depending on scarce labeled industrial data.
- **XGBoost for real-time classification** — low latency inference on minute-level feature vectors; no sequence modeling required since anomalies are point events.
- **Prophet for daily forecasting** — handles seasonality (weekends, month-end) and accepts exogenous regressors (production load, anomaly intensity) cleanly.
- **Three data granularities** — minute-wise (detection), batch-wise (operational review), daily (forecasting) each serve a distinct use case on the same dashboard.
- **Prescription mapping** — deterministic lookup table maps each anomaly class to a root cause and fix, making the system actionable without a separate recommendation engine.
