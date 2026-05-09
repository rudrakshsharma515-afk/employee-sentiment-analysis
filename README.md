# 🧠 Employee Sentiment Analysis — LLM Assessment Project

A production-ready pipeline that uses **Claude (Anthropic LLM)** to label employee feedback sentiments and derives actionable HR insights including monthly scoring, employee ranking, flight risk identification, and linear regression trend analysis.

---

## 📋 Table of Contents
- [Overview](#overview)
- [Project Structure](#project-structure)
- [Setup & Installation](#setup--installation)
- [Usage](#usage)
- [Methodology](#methodology)
- [Outputs](#outputs)
- [Dataset](#dataset)
- [Configuration](#configuration)

---

## Overview

This project implements a complete HR analytics pipeline on employee feedback data:

| Step | Module | Description |
|------|--------|-------------|
| 1 | `01_sentiment_labeling.py` | Label each feedback as Positive / Neutral / Negative using Claude API |
| 2 | `02_eda_visualization.py` | Exploratory Data Analysis with 6 charts |
| 3 | `03_monthly_scoring.py`   | Monthly sentiment scores per employee & department |
| 4 | `04_employee_ranking.py`  | Composite engagement score + tier classification |
| 5 | `05_flight_risk.py`       | Multi-signal flight risk flagging |
| 6 | `06_regression_model.py`  | Linear regression trends + 3-month forecast |

---

## Project Structure

```
employee-sentiment-analysis/
├── main.py                        # Orchestrates full pipeline
├── 01_sentiment_labeling.py       # LLM-based sentiment labeling
├── 02_eda_visualization.py        # EDA and charts
├── 03_monthly_scoring.py          # Monthly score computation
├── 04_employee_ranking.py         # Employee ranking & tiers
├── 05_flight_risk.py              # Flight risk identification
├── 06_regression_model.py         # Linear regression analysis
├── requirements.txt
├── .env.example
├── README.md
├── data/
│   ├── generate_sample_data.py    # Generates synthetic dataset
│   ├── employee_feedback.csv      # Input: raw feedback (your dataset)
│   ├── labeled_feedback.csv       # Output: sentiment-labeled data
│   ├── monthly_scores.csv         # Output: monthly scores
│   ├── employee_rankings.csv      # Output: employee rankings
│   ├── flight_risk_report.csv     # Output: flight risk report
│   └── regression_results.csv     # Output: regression results
└── outputs/
    ├── 01_sentiment_distribution.png
    ├── 02_sentiment_by_department.png
    ├── 03_monthly_sentiment_trend.png
    ├── 04_heatmap_dept_month.png
    ├── 05_sentiment_pie.png
    ├── 06_boxplot_dept.png
    ├── 07_monthly_overall_score.png
    ├── 08_monthly_dept_trends.png
    ├── 09_score_delta.png
    ├── 10_employee_ranking.png
    ├── 11_tier_distribution.png
    ├── 12_flight_risk_distribution.png
    ├── 13_flight_risk_scatter.png
    ├── 14_flight_risk_trends.png
    ├── 15_overall_regression.png
    ├── 16_per_employee_slopes.png
    └── 17_feature_regression.png
```

---

## Setup & Installation

### Prerequisites
- Python 3.9+
- pip
- An Anthropic API key (free tier works; or use the rule-based fallback)

### 1. Clone the repository
```bash
git clone https://github.com/your-username/employee-sentiment-analysis.git
cd employee-sentiment-analysis
```

### 2. Create and activate a virtual environment
```bash
python -m venv venv

# macOS / Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install -r requirements.txt
```

### 4. Configure environment variables
```bash
cp .env.example .env
# Open .env and add your ANTHROPIC_API_KEY
```

### 5. Prepare your dataset
Place your CSV file at `data/employee_feedback.csv`.

**Required columns:**

| Column | Type | Example |
|--------|------|---------|
| `Employee_ID` | string | `E001` |
| `Employee_Name` | string | `Alice Johnson` |
| `Department` | string | `Engineering` |
| `Date` | YYYY-MM-DD | `2024-03-15` |
| `Feedback` | string | `I really enjoy working here...` |

> **No dataset?** Run `python data/generate_sample_data.py` to create a realistic 180-row synthetic dataset.

---

## Usage

### Run the full pipeline (recommended)
```bash
python main.py
```

### Run individual steps
```bash
# Step 1: Sentiment labeling
python 01_sentiment_labeling.py

# Step 2: EDA & visualizations
python 02_eda_visualization.py

# Step 3: Monthly scoring
python 03_monthly_scoring.py

# Step 4: Employee ranking
python 04_employee_ranking.py

# Step 5: Flight risk identification
python 05_flight_risk.py

# Step 6: Linear regression
python 06_regression_model.py
```

### No API key? Use rule-based labeling
The pipeline automatically falls back to keyword-based sentiment labeling when no `ANTHROPIC_API_KEY` is set. You'll see:
```
No ANTHROPIC_API_KEY found — using rule-based labeling.
```

---

## Methodology

### 1. Sentiment Labeling
Each feedback text is sent to **Claude** (`claude-sonnet-4-20250514`) with a structured system prompt requesting exactly one label: `Positive`, `Neutral`, or `Negative`. Responses are parsed from JSON. A keyword-based fallback handles offline/no-key scenarios.

**Sentiment scores:** `Positive = +1`, `Neutral = 0`, `Negative = −1`

### 2. Exploratory Data Analysis
Six visualizations cover:
- Overall distribution (bar + pie)
- Sentiment by department (grouped bar)
- Monthly trend (stacked area)
- Department × Month heatmap (avg score)
- Score box plots per department

### 3. Monthly Sentiment Scoring
Monthly averages are computed per employee and per department. A **delta chart** compares the last 3 months vs. the full-year average, surfacing employees whose sentiment is shifting.

### 4. Employee Ranking
A **Composite Score** is computed as:
```
Composite Score = 0.6 × Full-Year Score + 0.4 × Recent 3-Month Score
```
Employees are ranked and assigned to engagement tiers:

| Tier | Score Range |
|------|-------------|
| High Engagement | ≥ 0.4 |
| Moderate Engagement | 0.0 – 0.4 |
| At Risk | −0.4 – 0.0 |
| Disengaged | ≤ −0.4 |

### 5. Flight Risk Identification
Three binary flags are checked:

| Flag | Condition |
|------|-----------|
| Low Score | Composite Score < −0.1 |
| Declining Trend | Monthly slope < −0.05 |
| High Negative Rate | Recent 3-month negative rate ≥ 45% |

Risk level is the count of flags triggered:
- **3 flags → Critical**, **2 → High**, **1 → Medium**, **0 → Low**

### 6. Linear Regression
Three regression analyses are run:
- **Overall trend:** Monthly avg score ~ time index, with 3-month forecast
- **Per-employee:** OLS slope per employee, classified as Improving / Stable / Declining
- **Feature-based:** Predicts score using department, month, and lagged score (train/test split, RMSE + R² reported)

---

## Outputs

All charts are saved to `outputs/` as high-resolution PNGs (150 DPI).
All computed data is saved to `data/` as CSVs.

### Key output files

| File | Description |
|------|-------------|
| `data/labeled_feedback.csv` | Original data + Sentiment label + Score |
| `data/monthly_scores.csv` | Per-employee monthly avg scores |
| `data/employee_rankings.csv` | Full ranking with composite score & tier |
| `data/flight_risk_report.csv` | Risk flags, slope, and risk level per employee |
| `data/regression_results.csv` | Per-employee regression slope, R², p-value |

---

## Dataset

The expected dataset is employee feedback data with at minimum:
`Employee_ID`, `Employee_Name`, `Department`, `Date`, `Feedback`

If you don't have the original dataset, the synthetic generator (`data/generate_sample_data.py`) creates a representative 15-employee, 12-month dataset with realistic sentiment trajectories including improving, stable, declining, and at-risk employees.

---

## Configuration

| Variable | Default | Description |
|----------|---------|-------------|
| `ANTHROPIC_API_KEY` | *(required for LLM mode)* | Your Anthropic API key |
| `DATASET_PATH` | `data/employee_feedback.csv` | Path to input CSV |
| `CLAUDE_MODEL` | `claude-sonnet-4-20250514` | Claude model string |

Thresholds (editable at top of each script):

| Parameter | Default | Location |
|-----------|---------|----------|
| Flight risk score threshold | `−0.1` | `05_flight_risk.py` |
| Flight risk slope threshold | `−0.05` | `05_flight_risk.py` |
| Flight risk negative rate | `0.45` | `05_flight_risk.py` |
| Regression significance level | `0.05` | `06_regression_model.py` |

---

## License
MIT — free to use and adapt.
