# DA6701 - Data Science for Finance Coursework

This repository contains the coursework and assignments for the **DA6701: Data Science for Finance** course, completed by **Team 14**. The project focuses on the application of machine learning, statistical regularization, and optimization techniques to financial data, ranging from stock price prediction to sophisticated portfolio management.

## 👥 Team 14 Members
- **Dharunpathi T** (ED23B018)
- **Pranav Raghuram** (ED23B054)
- **Sanjaith Ganeshkumar** (ED23B055)

---

## 📂 Repository Structure

The repository is organized into three main assignment modules, each tackling different aspects of financial analytics:

### 📊 Assignment 2: Financial Forecasting & Sentiment Analysis
Focuses on predicting market movements and analyzing public sentiment for a universe of stocks.
- **Key Features**:
  - Stock price predictions for NSE tickers.
  - Sentiment analysis of financial news/social data.
  - Initial portfolio optimization strategies.

### 📈 Assignment 3: Ex-Post ML Techniques for Portfolio Optimization
An in-depth study of statistical regularization and risk-based portfolio construction using a universe of 12 NSE stocks (TCS, INFY, HDFC Bank, etc.).
- **Key Features**:
  - **Ledoit-Wolf Shrinkage**: Improving the condition number of the sample covariance matrix to reduce instability in MPT.
  - **Markowitz Portfolio Theory (MPT)**: Implementation of Maximum Sharpe Ratio and Global Minimum Variance portfolios.
  - **Hierarchical Risk Parity (HRP)**: Advanced risk allocation through hierarchical clustering and recursive bisection.
  - **Visualization**: Covariance heatmaps, HRP dendrograms, and weight sensitivity analysis.

### 🎯 Assignment 4: Tracking Portfolio S&P 500
Implementation of various machine learning and heuristic models to replicate the performance of the S&P 500 index with a subset of stocks.
- **Key Features**:
  - **Autoencoders**: Using deep learning for dimensionality reduction and non-linear feature extraction for index tracking.
  - **Clustering**: Grouping stocks by performance characteristics to select representatives.
  - **Genetic Algorithms**: Evolutionary search for optimal portfolio weights.
  - **Lasso Regression**: Sparse model selection for effective portfolio replication.
  - **Greedy Algorithms**: Iterative selection of high-impact stocks.

---

## 🛠️ Tech Stack
- **Language**: Python (Jupyter Notebooks)
- **Data Source**: Yahoo Finance (`yfinance`)
- **Analysis**: Pandas, NumPy, Scipy
- **Machine Learning**: Scikit-learn, Autoencoders
- **Visualization**: Matplotlib, Seaborn

---

## 🚀 Key Technical Highlights
- **Risk Management**: Moving beyond mean-variance optimization to robust methods like HRP that do not require matrix inversion.
- **Index Replication**: Balancing the trade-off between tracking error and portfolio sparsity (number of stocks).
- **Statistical Fidelity**: Handling ill-conditioned financial data through shrinkage estimators (Ledoit-Wolf).
