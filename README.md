# Sales-Forecast
Time-series demand forecasting &amp; sales analytics using Python, Statsmodels, and Scikit-learn,
# Walmart Sales Forecasting

An end-to-end data science project that turns historical Walmart store sales data into validated insight, a tested time-series forecast, and a foundation for an interactive decision-support dashboard — built as part of the seKer AI Data Science Internship (Project 02).

---

## Table of Contents

- [Project Overview](#project-overview)
- [Business Problem](#business-problem)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Data Preparation](#data-preparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Key Findings](#key-findings)
- [Forecasting Methodology](#forecasting-methodology)
- [Model Evaluation](#model-evaluation)
- [Dashboard](#dashboard)
- [Business Recommendations](#business-recommendations)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Usage](#usage)
- [Screenshots](#screenshots)
- [Future Improvements](#future-improvements)
- [Author](#author)

---

## Project Overview

This project analyses historical weekly sales from 45 Walmart stores across multiple departments (2010–2012). It moves through the full data science workflow — data cleaning, exploratory analysis, statistical modelling, time-series forecasting, and honest out-of-sample evaluation — culminating in a baseline forecast that beats both a naive mean predictor and a standard linear regression on held-out data.

## Business Problem

Retail operations depend on accurate demand forecasts. Overestimating demand ties up capital in unsold inventory; underestimating it causes stockouts and lost revenue. This project asks: **can weekly sales be predicted reliably enough to inform inventory, staffing, and promotional decisions?**

## Objectives

- Understand and document a real-world retail sales dataset
- Clean and prepare raw, imperfect data
- Perform exploratory data analysis (EDA)
- Identify sales trends, seasonality, and holiday effects
- Analyse product (department), store-type, and regional performance
- Build and evaluate forecasting models
- Communicate findings through a business dashboard
- Produce evidence-backed business recommendations

## Dataset

| Field | Details |
|---|---|
| **Name** | Walmart Recruiting – Store Sales Forecasting |
| **Source** | [Kaggle Competition](https://www.kaggle.com/c/walmart-recruiting-store-sales-forecasting) |
| **Files** | `train.csv`, `features.csv`, `stores.csv` |
| **Time Period** | 2010-02-05 to 2012-10-26 (weekly) |
| **Records** | 421,570 rows after merge |
| **Licence** | Kaggle Competition Terms |

### Column Descriptions

| Column | Description |
|---|---|
| `Store` | Store number (1–45) |
| `Dept` | Department number |
| `Date` | Week of sale (weekly granularity) |
| `Weekly_Sales` | Sales for the given store–department–week |
| `IsHoliday` | Whether the week contains a holiday |
| `Temperature` | Average temperature in the region |
| `Fuel_Price` | Cost of fuel in the region |
| `CPI` | Consumer Price Index |
| `Unemployment` | Unemployment rate |
| `Type` | Store type (A, B, C) |
| `Size` | Store size in square feet |

> ⚠️ **MarkDown1–5 columns were dropped** — over 60% missing, not available before Nov 2011.

## Data Preparation

Documented cleaning decisions:

1. **Merged** `train`, `features`, and `stores` on `Store`/`Date`/`IsHoliday`.
2. **Dropped** MarkDown columns (majority missing).
3. **Converted** `Date` from object to datetime and set as index.
4. **Dropped** `Size` after initial correlation analysis.
5. **Converted** `IsHoliday` from bool → int for correlation.
6. **Aggregated** to weekly totals via `resample("W").sum()` for forecasting.

## Exploratory Data Analysis

Analysis performed:

- Total sales over time (weekly resampled, in millions)
- Year-over-year distributions (2010, 2011, 2012)
- December sales comparison across years
- Count of store types
- Total revenue by store type
- Top-performing departments
- Total revenue by holiday vs non-holiday
- Correlation of numeric features with `Weekly_Sales`
- Correlation heatmap
- Autocorrelation (ACF) and Partial Autocorrelation (PACF) plots

## Key Findings

| Finding | Evidence |
|---|---|
| **Type A stores dominate revenue** | $4,331M vs $2,001M (B) and $406M (C) |
| **Departments 92, 95, 38 lead sales** | $483M, $449M, $393M respectively |
| **Holiday weeks outsell non-holiday weeks** | Clear spike in the `IsHoliday` bar chart |
| **December shows strong seasonal lift** | Visible peaks in both 2010 and 2011 December plots |
| **Autocorrelation is weak at lag 1** | ACF shows little correlation — series is noisy |
| **No strong linear drivers** | Top correlation is `Dept` at only 0.148 |

## Forecasting Methodology

1. Aggregated `Weekly_Sales` to weekly totals → 143 observations.
2. Created lag-1 feature (`Weekly_Sales_L1`) → 142 observations.
3. **Train/test split:** 90% train (~127 weeks), 10% test (~15 weeks).
4. **Baseline:** predict the training mean.
5. **Model 1 — Linear Regression** using lag-1 as the only feature.
6. **Model 2 — AutoReg (lags=5)** on the raw weekly series.
7. **Walk-Forward Validation:** retrain AutoReg(5) for each test step, appending each actual observation before predicting the next.

## Model Evaluation

**Metric chosen:** MAE (Mean Absolute Error) — easy to explain to a business audience, robust to extreme values.

| Model | Train MAE | Test MAE |
|---|---|---|
| Baseline (mean) | 3.14 M | — |
| Linear Regression (lag-1) | 2.98 M | **1.30 M** |
| AutoReg(5) — static | 3.04 M | 1.70 M |
| **AutoReg(5) — Walk-Forward** | — | **1.41 M** |

> ✅ All models beat the baseline. Linear Regression performed best on the static split; Walk-Forward AutoReg produced more honest, sequential 1-step forecasts.

