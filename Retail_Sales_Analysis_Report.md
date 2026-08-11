# Retail Sales Analytics: End-to-End Analysis & Demand Forecast

**Dataset:** 3 years of daily point-of-sale data (Jan 2023 – Dec 2025) across 5 stores and 6 product categories, 32,880 store-category-day records.

---

## 1. Executive Summary

- **Total revenue:** $40.8M across 1,096 days, 1.28M units sold, average order value **$31.78**.
- **Growth:** revenue grew **+9.6% in 2024** and **+8.9% in 2025**, a consistent ~9% YoY trajectory.
- **Top performer:** Electronics leads all categories at **$10.6M** in revenue; Toys is the smallest category by total revenue but the most seasonally volatile (spikes ~40% above baseline in November–December).
- **Best store:** Store A (New York) generates the most revenue ($10.1M), consistent with it having the highest baseline traffic multiplier.
- **Promotions work:** promo days lift average units sold by **+46.6%** versus regular days.
- **Weekly pattern:** Saturday is the strongest day, Thursday the weakest.
- **Forecasting:** a Random Forest model predicts daily total revenue with **6.6% MAPE** (R² = 0.91), a **44% improvement** over a naive "same as last week" baseline.

---

## 2. Revenue Trend

![Revenue Trend](../charts/01_revenue_trend.png)

Daily revenue is noisy but the 30-day moving average makes the structure clear: a steady upward trend, a sharp Black Friday–Christmas spike each November/December (roughly 2× baseline), and smaller bumps around New Year's and July 4th. The pattern repeats consistently across all three years, which is a strong signal for any forecasting model to exploit.

## 3. Category Performance

![Revenue by Category](../charts/02_revenue_by_category.png)

Electronics dominates on total revenue despite lower unit volume, because of its high price point ($180 average vs. $12–60 for other categories). Groceries sells far more units but at low margin per unit. This split matters for strategy: Electronics offers revenue concentration risk, while Groceries offers volume stability.

## 4. Store Performance

![Revenue by Store](../charts/03_revenue_by_store.png)

Store A (New York) leads, followed by Store B (Los Angeles). The ranking is stable across the full period, suggesting the gap reflects structural differences (store size, local demand) rather than temporary factors — a useful segmentation for resource allocation or expansion planning.

## 5. Seasonality by Category

![Seasonality by Category](../charts/04_seasonality_by_category.png)

Normalizing each category's monthly revenue as a % of its own annual total reveals very different seasonal shapes:
- **Toys** are extremely concentrated in Nov/Dec (holiday gifting).
- **Groceries** are nearly flat year-round — a defensive, non-discretionary category.
- **Electronics and Apparel** show moderate holiday and back-to-school-style lifts.

This has direct implications for inventory planning: categories like Toys need aggressive pre-holiday stocking and post-holiday markdown planning, while Groceries can run lean, steady replenishment.

## 6. Weekly Pattern

![Weekday Pattern](../charts/05_weekday_pattern.png)

Weekend days (especially Saturday) outperform weekdays, with Thursday the weakest day of the week — a common retail pattern. Staffing and promotional timing could be aligned to this cycle (e.g., launching promotions Thursday to lift the slowest day).

## 7. Promotion Effectiveness

![Promo Impact](../charts/06_promo_impact.png)

Promotional days show a **46.6% lift** in average units sold relative to regular days, even after accounting for the price discount applied during promotions. This confirms promotions are an effective lever, though the analysis doesn't yet isolate margin impact — a next step would be comparing promo *profit* (not just units) against non-promo days.

## 8. Correlation Structure

![Correlation Heatmap](../charts/07_correlation_heatmap.png)

Revenue correlates most strongly with units sold (expected, since revenue = units × price). Promotions correlate positively with units sold but are mildly negatively correlated with unit price, reflecting the built-in discount — consistent with the promo-lift finding above.

---

## 9. Predictive Model: Daily Revenue Forecast

**Goal:** predict next-day total revenue using calendar features, holiday windows, promo activity, and lagged/rolling revenue history.

**Method:** Random Forest Regressor, trained on the first ~2.7 years of data and evaluated on a held-out final 90 days (a time-based split, which avoids leaking future information into training — the correct approach for time series, unlike a random train/test split).

![Forecast vs Actual](../charts/08_forecast_actual_vs_predicted.png)

| Metric | Random Forest | Naive Baseline (same day last week) |
|---|---|---|
| MAE | $3,338 | $6,010 |
| MAPE | 6.6% | 12.4% |
| R² | 0.91 | — |

The model cuts forecast error nearly in half compared to the naive baseline, and tracks the holiday spike in the test window closely.

![Feature Importance](../charts/09_feature_importance.png)

**What drives the forecast:**
1. **Revenue 7 days ago** (58% importance) — the dominant signal, confirming strong weekly seasonality.
2. **Holiday window flag** (14%) — captures the Nov/Dec surge.
3. **7-day rolling average** (11%) — smooths recent momentum.
4. Day-of-week and weekend flags contribute smaller but meaningful signal.

---

## 10. Conclusions & Recommendations

1. **Inventory & staffing** should be built around the weekly cycle (Saturday peak, Thursday trough) and the Nov–Dec holiday surge, which is large and highly predictable.
2. **Category strategy should diverge**: protect and expand Electronics (revenue driver), keep Groceries lean and steady (non-seasonal, high volume), and plan Toys inventory specifically around the holiday window with clear post-holiday markdown timing.
3. **Promotions are effective** at driving unit volume (+46.6%) — worth extending, but future work should evaluate promotion ROI on a *margin* basis, not just unit lift.
4. **Store A and B are the highest-value locations** and reasonable candidates for benchmarking best practices against lower-performing stores (C–E).
5. **The forecasting model is production-viable as a first pass** (6.6% MAPE) for short-term (next-day/next-week) revenue planning, and could be extended with external features (weather, local events, marketing spend) for further gains.

---

## 11. Methodology Notes & Limitations

- This dataset was **synthetically generated** to mimic realistic retail POS patterns (trend, weekly/yearly seasonality, holiday effects, promotions, noise) for the purposes of this end-to-end demonstration project. The modeling techniques and code apply directly to real POS/ERP exports (Square, Shopify, SAP, etc.) with the same or similar schema (date, store, category, units, price, revenue).
- The forecasting model uses a **time-based** (not random) train/test split, which is the correct validation strategy for time series — random splits would leak future information and overstate accuracy.
- Feature importance reflects a single Random Forest run; a production system should validate with cross-validation across multiple time windows and check for feature stability.
