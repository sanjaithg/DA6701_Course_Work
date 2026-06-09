# Sentiment-Aware Multi-Source Stock Return Prediction & Portfolio Construction

A predictive system for a six-stock Indian equity universe — **RELIANCE, HDFCBANK, INFY,
MM (Mahindra), BHARTIARTL, HUL** — that forecasts daily returns from multiple information
sources and turns the forecasts into a forward-looking portfolio.

The pipeline fuses structured market data, firm fundamentals, macroeconomic indicators,
capital flows, commodities, FX, volatility indices, and **financial-news sentiment** (GDELT
articles scored with FinBERT) into a single machine-learning framework. Everything is built
under realistic constraints: temporal consistency, no look-ahead bias, and robustness across
all six securities.

## Repository Layout

```
Sentiment-Aware-Stock-Return-Prediction/
├── Sentiment Analysis/                 # FinBERT news-sentiment scoring (feeds the predictors)
│   └── Assignment2_Sentiment_FinBERT.ipynb
├── Approach 1 - Broad Numeric Pool/    # Feature set = broad numeric pool + fundamentals added back
│   ├── RELIANCE_…  HDFCBANK_…  INFY_…  MM_…  BHARTIARTL_…  HUL_Stock_Prediction_A1.ipynb
├── Approach 2 - Curated Technical Pool/# Feature set = curated always-keep vars + technical FS pool
│   ├── RELIANCE_…  HDFCBANK_…  INFY_…  MM_…  BHARTIARTL_Stock_Prediction_A2.ipynb
└── Portfolio Optimization/             # Efficient-frontier / tangency portfolio from the forecasts
    └── Portfolio_Optimisation.ipynb
```

There is **one notebook per stock per approach** because the full feature-engineering,
selection, tuning, and evaluation pipeline is run independently for each security.

## Data Sources

| Source | Features | Frequency |
|--------|----------|-----------|
| Yahoo Finance | OHLCV, NIFTY50 return, sector return | Daily |
| Computed | log return, volume change, technical indicators | Daily |
| Perplexity Finance | EPS, PE, revenue, net income, profit margin, … | Quarterly |
| NSE | India VIX | Daily |
| RBI | repo rate, 91d T-bill, 10y G-sec yield, yield-curve spread, CPI, CG/SG securities | Daily–Monthly |
| Trendlyne | FII / DII net flows | Monthly |
| RBI | USD/EUR/GBP/JPY-INR | Daily |
| Investing.com | crude (Brent) return, gold return | Daily |
| GDELT + FinBERT | daily news sentiment score | Daily |

## Procedure

1. **Sentiment scoring** (`Sentiment Analysis/`). Filter GDELT news to English, score each
   article with **ProsusAI/FinBERT** (positive / negative / neutral), and aggregate to a
   continuous daily `sentiment_score` per stock — the textual feature consumed downstream.
2. **Preprocessing.** Drop features with >15% missing; forward-fill where temporally
   appropriate; compute log returns; winsorize at the 1% tails; temporally shift predictors
   so each strictly precedes the target. **All transforms are fit on train only.**
3. **Feature engineering.** Replace raw price levels with stationary technical indicators
   (SMA/EMA, MACD, RSI, Bollinger bands, ATR, OBV, volume z-score, lagged returns/volume),
   pairwise interactions among top-MI predictors, and 30-day rolling standardization with a
   one-period shift.
4. **Two feature-pool designs.**
   - *Approach 1 — Broad Numeric Pool:* all numeric columns (minus meta/target/leaky/
     redundant) compete in selection, with core fundamentals added back.
   - *Approach 2 — Curated Technical Pool:* fundamentals/macro/flows/FX/commodities/sentiment
     are *always kept*; selection is applied mainly to engineered technical & lag features.
5. **Feature selection (both approaches).** Four methods — Domain Mutual Information,
   Random-Forest importance (90% cumulative), RFE with Ridge, and SHAP (LightGBM) — combined
   by **majority vote** (a feature is kept if ≥2 methods select it).
6. **Modeling.** Tree ensembles (LightGBM, XGBoost, Random Forest, CatBoost) in both
   **regression** (residualized return) and **classification** (return direction) form,
   tuned by grid search on a 70/15/15 train/val/test split; boosting uses 1000 rounds with
   30-round early stopping. Selection by min validation RMSE (regressors) / log-loss
   (classifiers); reported metrics are on the strictly unseen test set.
7. **Portfolio construction** (`Portfolio Optimization/`). Feed the forecasts into a Monte
   Carlo / mean–variance optimizer; the tangency portfolio (max Information Ratio on the
   efficient frontier) is selected and backtested out-of-sample.

## Results

> All numbers below are taken directly from the notebook output cells. Across both
> approaches, regression R² on residualized daily returns is essentially zero (often
> slightly negative) — daily return level is hard to predict — while **direction** is more
> learnable. CatBoost is the most consistent regressor; tree ensembles reach 50–64%
> directional accuracy.

### Sentiment Analysis (FinBERT) — per-stock daily sentiment

| Stock | English articles | Days covered | Avg sentiment |
|-------|-----------------:|-------------:|--------------:|
| RELIANCE | 10,937 | 409 | 0.097 |
| HDFCBANK | 10,864 | 506 | 0.042 |
| INFY | 12,214 | 462 | 0.055 |
| BHARTIARTL | 10,423 | 599 | 0.140 |
| HUL | 10,958 | 1,129 | 0.073 |
| MM | 12,240 | 251 | 0.099 |

All stocks score mildly positive overall (BHARTIARTL most positive, HDFCBANK least).

### Approach 1 — test-set metrics (best model per stock)

| Stock | Best regressor (MAE / R²) | Best classifier (Acc / HC-Acc) |
|-------|---------------------------|--------------------------------|
| RELIANCE | CatBoost 0.00859 / 0.0193 | RF 50.7% / 60.5% |
| HDFCBANK | CatBoost 0.00728 / 0.0111 | RF 48.4% / 30.0% |
| INFY | XGB 0.00858 / 0.0532 | RF 54.0% / 40.0% |
| MM | CatBoost 0.01185 / 0.0139 | RF 54.0% / 60.0% |
| BHARTIARTL | CatBoost 0.01146 / 0.0305 | RF 64.4% / LGBM HC 71.4% |
| HUL | CatBoost 0.00819 / -0.0008 | RF 56.1% / 57.8% |

BHARTIARTL is the standout (directional accuracy up to 64.4%, high-confidence 71.4%).

### Approach 2 — test-set metrics (best model per stock; 5 stocks, no HUL)

| Stock | Best regressor (MAE / R²) | Best classifier (Acc / HC-Acc) |
|-------|---------------------------|--------------------------------|
| RELIANCE | CatBoost 0.00869 / 0.0164 | XGB 54.0% / 46.5% |
| HDFCBANK | CatBoost 0.00706 / -0.0006 | LGBM/XGB 50.5% / 49.5% |
| INFY | LGBM 0.01123 / 0.0313 | RF 57.4% / 68.2% |
| MM | CatBoost 0.01186 / 0.0093 | RF 53.5% / 62.8% |
| BHARTIARTL | XGB 0.00945 / -0.0102 | XGB 53.7% / 45.5% |

INFY classifiers are strongest in Approach 2 (Acc 57.4%, high-confidence 68.2%).

### Portfolio Optimization

Average optimal (tangency) weights, backtested over 62 trading days (Oct–Dec 2025):

| Stock | Weight |
|-------|-------:|
| INFY | 22.02% |
| HDFCBANK | 21.89% |
| BHARTIARTL | 20.84% |
| MM | 14.49% |
| RELIANCE | 13.23% |
| HUL | 7.53% |

*(The notebook renders Annualized Return, Volatility, Information Ratio, Max Drawdown, and
Directional Hit Ratio inside the equity-curve plot rather than as printed text; the
allocation and 62-day backtest window are the textual outputs.)*

## Key Takeaways

- Daily **return level** is near-unpredictable (R² ≈ 0); **direction** is more tractable, so
  classification is the more reliable formulation here.
- Boosting models (XGBoost / CatBoost) generalize more consistently than Random Forest in
  regression; tree ensembles dominate the classification leaderboards.
- Predictive signal comes from a *mix* of technical, lagged, macro, and fundamental features
  plus news sentiment — no single group dominates.
- Even with modest per-stock signal, the portfolio step translates the forecasts into a
  diversified, economically sensible allocation.
