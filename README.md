# ⚾ Cincinnati Reds Home Attendance Prediction

**Predictive modeling of Cincinnati Reds home game attendance at Great American Ballpark using historical schedule data, team performance metrics, and weather conditions (2015–2024).**

> DSC 630 — Predictive Analytics | Bellevue University | Professor Toni Farley | By Matthew Rice

---

## 📋 Table of Contents

- [Project Overview](#project-overview)
- [Business Context](#business-context)
- [Dataset](#dataset)
- [Data Pipeline](#data-pipeline)
- [Models](#models)
- [Results](#results)
- [Visualizations](#visualizations)
- [Recommendations](#recommendations)
- [Repository Structure](#repository-structure)
- [Requirements](#requirements)
- [How to Run](#how-to-run)
- [References](#references)

---

## Project Overview

This project builds a regression-based attendance forecasting model for Cincinnati Reds home games at Great American Ballpark. Using ten seasons of game-level data (2015–2024), the goal is to identify which factors most influence fan turnout and produce a model capable of predicting attendance in advance — giving ownership and management a tool to optimize ticket sales, pricing, and promotional strategy.

**Primary research question:** *What factors predict home game attendance for the Cincinnati Reds, and how accurately can attendance be forecasted before a game is played?*

---

## Business Context

Ballpark revenue — seating, concessions, and parking — accounted for 41% of total MLB revenue in 2023, a higher proportion than either the NFL or NBA (Visual Capitalist, 2024). For a small-market team like the Reds, maximizing per-game attendance is critical. The most common games at Great American Ballpark draw only 10,000–20,000 fans against a capacity of over 40,000, representing significant unrealized revenue.

A reliable attendance model enables:
- Targeted promotions on games predicted to underperform
- Data-driven ticket pricing adjustments
- Strategic scheduling of giveaways and special events
- More accurate revenue forecasting for operations planning

---

## Dataset

### Primary Source
**`Reds_Schedule.xlsx`** — A multi-sheet Excel file containing Cincinnati Reds game-by-game schedule and results data from 2015 to 2024, one season per sheet. Fields include:

| Feature | Description |
|---|---|
| `Date` | Game date |
| `Opp` | Opponent team |
| `W/L` | Game result (Win/Loss) |
| `Attendance` | Reported home game attendance |
| `Streak` | Team's current winning/losing streak (+/−) |
| `Home_Away` | Home or away designation |

### Engineered Features

| Feature | Description |
|---|---|
| `win_pct` | Cumulative winning percentage at time of game |
| `streak_int` | Streak converted from +/− symbols to signed integers |
| `weekday` | Day of the week the game was played |
| `Month` | Calendar month of the game |
| `temp` | Average game-day temperature (°F) from OpenWeatherMap API |
| `humidity` | Average game-day humidity from OpenWeatherMap API |
| `weather_description` | Most frequent weather condition label for the game day |

### Data Notes
- **2020 excluded:** No fans attended games due to COVID-19; these rows are dropped from modeling
- **Winning percentage filtered:** Records with `win_pct` outside 0.15–0.80 removed as extreme outliers
- **Categorical encoding:** Opponent, month, weather description, and weekday one-hot encoded via `pd.get_dummies()`
- **Weather data:** Retrieved via [OpenWeatherMap Historical API](https://openweathermap.org/history) using Great American Ballpark coordinates (39.0974°N, 84.5071°W)

---

## Data Pipeline

```
Reds_Schedule.xlsx (10 sheets, 2015–2024)
        │
        ▼
Load & concatenate all seasons → combined DataFrame
        │
        ▼
Filter home games only
        │
        ▼
Calculate cumulative win% and streak (integer)
        │
        ▼
Query OpenWeatherMap API for each game date
        │
        ▼
Add weekday and month features
        │
        ▼
Drop 2020 (COVID), drop irrelevant columns
        │
        ▼
One-hot encode categoricals → model-ready DataFrame
        │
        ▼
Train/Test split (80/20) → Regression modeling
```

---

## Models

The project evaluates the following regression approaches, assessed using **RMSE** (primary) and **R²**:

| Model | Purpose |
|---|---|
| **Multiple Linear Regression** | Baseline; interpretable coefficient analysis |
| **Ridge Regression** | Handles multicollinearity among weather/date features |
| **Lasso Regression** | Automatic feature selection via L1 regularization |
| **Random Forest Regression** | Captures non-linear interactions; feature importance output |
| **Gradient Boosting / XGBoost** | Strong performance on tabular data with complex patterns |

All models use a consistent 80/20 train-test split with k-fold cross-validation (5 or 10 folds) to assess generalizability.

---

## Results

**Baseline Linear Regression (current milestone):**

| Metric | Value |
|---|---|
| R² | 0.422 |
| Interpretation | Model explains ~42% of variance in attendance |

Residual analysis revealed a **right-skewed distribution**, suggesting influential outliers (e.g., Opening Day, marquee matchups) are affecting the model. Next steps include log-transforming the attendance variable, scaling features, and testing regularized and ensemble models.

---

## Visualizations

The notebook produces the following charts:

| Chart | Description |
|---|---|
| **Attendance Histogram** | Distribution of home game attendance across all seasons |
| **Attendance by Opponent** | Scatter plot revealing which opponents are the strongest draws |
| **Average Attendance by Month** | Seasonality patterns; March skewed by Opening Day crowds |
| **Average Attendance by Year** | Long-term trends; 2020 gap; post-COVID recovery visible |
| **Winning Percentage Box Plot** | Distribution of team win% across all home games |
| **Residual Distribution** | Linear regression residual histogram with KDE overlay |

---

## Recommendations

- **Add promotional data:** Fireworks nights, bobblehead giveaways, and discounted ticket promotions likely drive significant unmodeled attendance variance. Manual collection from the team's website is the most feasible path.
- **Consider a 5-season window:** Limiting training data to 2019–2024 may improve model relevance by reducing the influence of outdated fan behavior patterns.
- **Log-transform attendance:** The right-skewed residuals suggest the target variable may benefit from transformation before modeling.
- **Frame as a guidance tool:** A model predicting within ~2,000–3,000 fans of actual attendance still provides actionable insight for a 40,000-seat ballpark.

---

## Repository Structure

```
Cincinnati-Reds-Home-Attendance-Prediction/
│
├── notebooks/
│   └── Red_Home_Attendance_Predition.ipynb   # Full pipeline and modeling notebook
│
└── README.md
```

> **Note:** `Reds_Schedule.xlsx` and intermediate CSV outputs are not included in this repository. Update the local file paths in the notebook before running. See [How to Run](#how-to-run) below.

---

## Requirements

```
pandas
numpy
matplotlib
seaborn
scikit-learn
requests
openpyxl
```

Install all dependencies:
```bash
pip install pandas numpy matplotlib seaborn scikit-learn requests openpyxl
```

An **OpenWeatherMap API key** is required to re-run the weather data collection step. Free keys (with historical data access) are available at [openweathermap.org](https://openweathermap.org/api).

---

## How to Run

1. Clone the repository:
```bash
git clone https://github.com/Bizarroxela/Red-Home-Attendance-Prediction.git
cd Red-Home-Attendance-Prediction
```

2. Place `Reds_Schedule.xlsx` in your local downloads or documents folder and update the path in the notebook:
```python
excel_file = '/your/path/Reds_Schedule.xlsx'
```

3. Add your OpenWeatherMap API key:
```python
API_KEY = 'your_api_key_here'
```

4. Launch the notebook:
```bash
jupyter notebook notebooks/Reds_Home_Attendance_Prediction.ipynb
```

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/Bizarroxela/Cincinnati-Reds-Home-Attendance-Prediction/blob/main/notebooks/Cincinnati-Red_Home_Attendance_Predition.ipynb)

---

## References

Visual Capitalist. (2024). *Visualized: How U.S. Sports Leagues Make Money.* Retrieved from https://www.visualcapitalist.com/u-s-sports-leagues-by-revenue/

---

## Author

**Matthew Rice** — [@Bizarroxela](https://github.com/Bizarroxela)

*Part of an applied data science portfolio covering sports analytics, healthcare analytics, machine learning, and data visualization.*
