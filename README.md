# Smart Attendance Tracker System

A fingerprint-biometric attendance system built on IoT sensor hardware, Firebase, and a Python data science pipeline. Raw fingerprint scan events are cleaned, feature-engineered, and scored for anomalies before being surfaced on a Power BI dashboard.

Currently built and validated on **synthetic employee attendance data** structured to match the expected IoT/Firebase output format. The same pipeline is intended to be reused for **student attendance data** once the IoT team completes hardware deployment for that use case.

## Architecture

```
Fingerprint Sensor → ESP32 / IoT Device → Firebase Database
                                                 │
                                                 ▼
                                    DS Pipeline (Python)
                          Cleaning → EDA → Feature Engineering
                                                 │
                                                 ▼
                                  Processed Data (CSV)
                                                 │
                                                 ▼
                                    Power BI Dashboard
```

## Project Structure

```
Smart_Attendance_Tracker_System/
├── .gitignore
├── README.md
├── configs/             # Runtime configuration and credentials (see Configuration below)
├── data/
│   ├── raw/              # Untouched source data (Smart_A_S_D.xlsx, Smart_A_S_D.csv)
│   └── proccesed/         # Pipeline outputs: cleaned_data.csv, featured_data.csv, employee_daily.csv
├── docs/
│   ├── Smart_Attendance_System_DS_Technical_Report.docx
│   └── Data_Inspection_Report_Device_Log.docx
├── models/              # Trained model artifacts (planned)
├── notebook/
│   ├── data_cleaning.ipynb        # Raw data validation, context-aware imputation, outlier review
│   ├── eda.ipynb                  # Attendance, biometric, and device-health exploratory analysis
│   └── feature_engineering.ipynb  # Event-level and employee-day-level feature construction
├── reports/             # Exported charts and dashboard exports (planned)
├── src/                 # Reusable Python modules (planned)
└── tests/               # Validation checks (planned)
```

## Pipeline

Run the notebooks in `notebook/` in order:

1. **`data_cleaning.ipynb`** — reads `data/raw/Smart_A_S_D.xlsx`, investigates *why* each column has missing values (rather than defaulting to blanket mode-fill), imputes accordingly, converts `timestamp` to datetime, reviews outliers, and writes `data/proccesed/cleaned_data.csv`.
2. **`eda.ipynb`** — attendance trends, chronic-absenteeism analysis, fingerprint match reliability, and device/IoT health analysis on the cleaned data.
3. **`feature_engineering.ipynb`** — builds two output tables from `cleaned_data.csv`:
   - `featured_data.csv` — event-level data with biometric reliability, device health, and behavioral anomaly features, plus a composite `anomaly_score`
   - `employee_daily.csv` — one row per employee per day, with `total_hours_present`, rolling attendance counts, absent streaks, and cumulative `attendance_percentage`

Both output CSVs are designed to be imported directly into Power BI (joined on `employee_id` + `date`).

## Documentation

Full technical writeups live in `docs/`:
- **`Smart_Attendance_System_DS_Technical_Report.docx`** — end-to-end DS pipeline report: architecture, dataset overview, cleaning methodology, EDA findings, feature engineering details, and current project status.
- **`Data_Inspection_Report_Device_Log.docx`** — device/log-level data inspection notes.

## Configuration

`.gitignore` sits at the project root (not inside `configs/` — git resolves ignore patterns relative to the file's own location, so it has to live at the top level to cover `data/`, `models/`, etc.). It excludes raw/processed data, Firebase credentials, and standard Python/Jupyter artifacts (`__pycache__`, `.ipynb_checkpoints`, virtual environments) from version control.

**Planned, not yet added:**
- `configs/config.yaml` — will centralize tunable thresholds (absentee risk cutoff, anomaly z-score threshold, rapid-repeat window, device health/signal thresholds) so they can be adjusted without editing notebook code.
- `configs/firebase_config.json` — real Firebase service-account credentials (gitignored, never committed). A `firebase_config.example.json` placeholder template will be committed in its place so anyone cloning the repo knows what fields are needed.

## Current Status

| Item | Status |
|---|---|
| Data Cleaning | Completed |
| Exploratory Data Analysis | Completed |
| Feature Engineering | Completed |
| Documentation (Technical Report, Data Inspection Report) | Completed |
| Project Structure & Config | In Progress (`.gitignore` added at root; `config.yaml` pending) |
| Power BI Dashboard | Pending |
| Firebase Live Ingestion | Pending (hardware/IoT dependent) |
| Anomaly Detection Model (ML-based) | Pending — rule-based `anomaly_score` in place as interim solution |
| Absenteeism Prediction Model | Pending |
| Student Data Migration | Pending (awaiting IoT team handoff) |

## Setup

```bash
git clone <repo-url>
cd Smart_Attendance_Tracker_System
pip install pandas openpyxl matplotlib seaborn
```

Open `notebook/data_cleaning.ipynb` in VS Code / Jupyter and run all cells, then `eda.ipynb`, then `feature_engineering.ipynb`, in that order.

## Notes

- All feature-engineering logic is keyed off `event_type`, `fingerprint_match`, and each entity's own historical behavior rather than employee-specific column values, so it is expected to carry over directly once student attendance data replaces the synthetic employee dataset.
- The `data/proccesed/` folder name is a known typo (should be `processed`) but is used consistently across all notebooks — rename only if updating every path reference at the same time.