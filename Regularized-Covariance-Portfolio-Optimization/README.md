# Regularized-Covariance Portfolio Optimization — Ledoit-Wolf Shrinkage & HRP

An empirical study of the instability of the sample covariance matrix in Markowitz
optimization on a **12-stock NSE universe** — TCS, INFY, HCLTECH, TATASTEEL, HINDALCO,
JINDALSTEL, HDFCBANK, ICICIBANK, KOTAKBANK, HINDUNILVR, ITC, NESTLEIND — and how two
regularization techniques fix it:

1. **Statistical shrinkage** — Ledoit-Wolf (LW), which conditions the covariance matrix so
   it can be inverted safely.
2. **Structural regularization** — Hierarchical Risk Parity (HRP), which avoids matrix
   inversion entirely by allocating risk across a learned cluster tree.

The two are stress-tested through bootstrap conditioning analysis, turnover measurement, and
a 2023 SVB banking-crisis weight-sensitivity case study.

## Notebook

| File | Description |
|------|-------------|
| `Regularization_LedoitWolf_HRP.ipynb` | Full study: sample-covariance conditioning, Ledoit-Wolf shrinkage, HRP recursive bisection, turnover, and crisis analysis |

Data span: 1,237 trading days (Mar 2021 – Feb 2026). Train: Apr 2021 – Feb 2025 (47 months /
984 days). Test: Mar 2025 – Feb 2026 (252 days).

## Procedure

1. **Diagnose ill-conditioning.** Compute the condition number `κ(S) = λ_max/λ_min` of the
   sample covariance and run a 300-resample bootstrap to show the instability is structural,
   not a finite-sample fluke.
2. **Ledoit-Wolf shrinkage.** Solve `min E‖Σ_LW − Σ‖²_F` with `Σ_LW = (1−α)S + αF` and target
   `F = μI`; the optimal `α` is the ratio of sample estimation error to sample–target
   misalignment. Re-examine the eigenvalue spectrum and condition number after shrinkage, and
   compare Max-Sharpe allocations Sample-MPT vs LW-MPT.
3. **Hierarchical Risk Parity.** Convert correlations to the distance `D_ij = sqrt(0.5(1−ρ_ij))`,
   cluster with Ward linkage, quasi-diagonalize the covariance, and allocate capital by
   recursive bisection (inverse-variance split `α_L = V_R/(V_L+V_R)`). Compare HRP vs
   Min-Variance MPT vs Equal-Weight.
4. **Turnover analysis.** Measure average one-way turnover under monthly rebalancing over a
   252-day window for MPT, LW-MPT, and HRP.
5. **Crisis case study.** Track banking-stock weight sensitivity `σ_w` over a 12-month rolling
   window through the 2023 SVB stress period for all three methods.

## Results

> All numbers below are taken directly from the notebook output cells.

### Part 1 — Sample covariance conditioning & Ledoit-Wolf

| Metric | Sample | LW-shrunk |
|--------|-------:|----------:|
| Condition number κ | 35.60 | 14.96 (**2.38× better**) |
| Max eigenvalue λ₁ | 0.3249 | 0.2749 (compressed) |
| Min eigenvalue λₙ | 0.0091 | 0.0184 (lifted) |

Bootstrap κ (300 resamples): mean **82.2**, std 31.7, max **222.9** — confirming structural
instability. Optimal shrinkage intensity **α = 0.1698 (16.98%)**.

**Max-Sharpe allocations (Sample-MPT vs LW-MPT):**

| Stock | Sample MPT | LW-MPT |
|-------|-----------:|-------:|
| TCS | 20.39% | 21.47% |
| HINDALCO | 34.39% | 31.33% |
| HDFCBANK | 31.87% | 31.20% |
| ICICIBANK | 13.35% | 16.01% |
| Other 8 assets | 0.00% | 0.00% |

| Portfolio metric | Sample MPT | LW-MPT |
|------------------|-----------:|-------:|
| HHI (concentration) | 0.2792 | 0.2672 |
| Annualized return | 22.55% | 22.79% |
| Annualized volatility | 13.70% | 13.64% |
| Sharpe | 1.645 | 1.671 |

### Part 2 — Hierarchical Risk Parity

The algorithm discovered **4 data-driven clusters** (independent of GICS sectors):

| Cluster | Members | Mean Vol | Mean Ret | Intra-ρ̄ |
|---------|---------|---------:|---------:|--------:|
| C1 | HCLTECH, ICICIBANK, ITC | 35.4% | 26.5% | 0.724 |
| C2 | TCS, JINDALSTEL, NESTLEIND | 22.8% | 16.1% | 0.664 |
| C3 | INFY, HINDALCO, KOTAKBANK | 22.2% | 10.5% | 0.498 |
| C4 | TATASTEEL, HDFCBANK, HINDUNILVR | 19.8% | 13.3% | 0.366 |

**Allocations (HRP vs Min-Var MPT vs Equal-Weight):**

| Stock | HRP | Min-Var MPT | Equal Wt |
|-------|----:|------------:|---------:|
| TCS | 9.99% | 4.84% | 8.33% |
| INFY | 8.31% | 10.69% | 8.33% |
| HCLTECH | 4.39% | 0.00% | 8.33% |
| TATASTEEL | 10.68% | 9.99% | 8.33% |
| HINDALCO | 8.23% | 10.66% | 8.33% |
| JINDALSTEL | 5.85% | 0.69% | 8.33% |
| HDFCBANK | 16.06% | 16.51% | 8.33% |
| ICICIBANK | 2.10% | 1.20% | 8.33% |
| KOTAKBANK | 10.91% | 5.78% | 8.33% |
| HINDUNILVR | 13.11% | 24.27% | 8.33% |
| ITC | 2.72% | 0.00% | 8.33% |
| NESTLEIND | 7.66% | 15.35% | 8.33% |
| **HHI** | **0.1023** | 0.1484 | 0.0833 |

HRP eliminates the 0%-weight dropouts (HCLTECH, ITC) that Min-Variance MPT forces.

### Turnover (average one-way, monthly rebalancing)

| Strategy | Turnover | Reduction vs MPT |
|----------|---------:|-----------------:|
| Min-Var MPT | 15.46% | — |
| LW-MPT | 13.75% | 11.1% |
| HRP | **9.11%** | **41.1%** |

### Crisis (2023 SVB) — banking-stock weight sensitivity σ_w

| Stock | Standard MPT | LW-MPT | HRP |
|-------|-------------:|-------:|----:|
| HDFCBANK | 35.52% | 30.83% | **4.90%** |
| ICICIBANK | 15.84% | 11.31% | **2.70%** |
| KOTAKBANK | 8.15% | 8.67% | **3.71%** |

## Key Takeaways

- Unregularized Markowitz is an "error maximizer": a high condition number (κ ≈ 36, bootstrap
  up to 223) amplifies estimation noise into extreme, concentrated weights.
- **Ledoit-Wolf** physically conditions the matrix (κ → 15), enabling safe inversion and
  cutting turnover modestly while slightly improving Sharpe.
- **HRP** bypasses inversion entirely: it removes corner solutions, lowers concentration
  (HHI 0.148 → 0.102), slashes turnover by 41%, and stays remarkably stable through the SVB
  crisis (HDFCBANK σ_w 35.5% → 4.9%). Regularization is a prerequisite for real-world
  trading.
