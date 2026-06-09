# DA6701 — Data Science & AI in Finance

Coursework for **DA6701: Data Science & AI in Finance** by **Team 14**
(Dharunpathi T · ED23B018 — Pranav Raghuram · ED23B054 — Sanjaith Ganeshkumar · ED23B055).

A collection of finance projects covering portfolio construction, covariance regularization,
machine-learning return prediction, index replication, and goal-based investing. Each project
is self-contained in its own folder with a write-up (`README.md`) and the Jupyter notebooks
that produced the results.

## Projects

| Project | Problem |
|---------|---------|
| [Mean-Variance-Portfolio-Optimization](Mean-Variance-Portfolio-Optimization/) | Build a 10-asset long-only portfolio from a 30-stock NSE universe via Modern Portfolio Theory, comparing greedy least-correlated and Hierarchical Risk Parity (HRP) asset selection with Monte-Carlo efficient frontiers. |
| [Sentiment-Aware-Stock-Return-Prediction](Sentiment-Aware-Stock-Return-Prediction/) | Forecast daily returns for six Indian stocks from market, fundamental, macro, and FinBERT news-sentiment data, then build a forward-looking portfolio. |
| [Regularized-Covariance-Portfolio-Optimization](Regularized-Covariance-Portfolio-Optimization/) | Quantify the instability of sample covariance in Markowitz optimization and fix it with Ledoit-Wolf shrinkage and HRP, stress-tested through turnover and the 2023 SVB crisis. |
| [Sparse-Index-Replication](Sparse-Index-Replication/) | Replicate the S&P 500 with ≤50 stocks using five methods — LASSO, autoencoder, genetic algorithm, greedy selection, and clustering — minimizing tracking error under a cardinality constraint. |
| [Goal-Based-Portfolio-Optimization](Goal-Based-Portfolio-Optimization/) | Find the static allocation across five Indian equities that maximizes the probability of hitting a ₹1.5 Cr retirement goal via Monte-Carlo simulation, under two liability schedules. |

## Repository Structure

Each project folder contains:
- a `README.md` describing the **problem, procedure, and results** (results taken directly
  from the notebook outputs), and
- the **Jupyter notebooks** and supporting files.

## Tech Stack

Python · NumPy · pandas · scikit-learn · SciPy · LightGBM · XGBoost · CatBoost · PyTorch
(FinBERT) · matplotlib — across Jupyter notebooks.
