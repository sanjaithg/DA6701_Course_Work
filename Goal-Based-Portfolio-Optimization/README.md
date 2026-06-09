# Goal-Based Portfolio Optimization via Monte Carlo

A goal-based investing problem. A client saves **₹20,000 per month** (with a **4% annual
step-up**) over a **20-year (240-month)** horizon and wants to retire with a terminal corpus
of **₹1.5 Cr**. Three intermediate goals must be met along the way; any shortfall is borrowed
at a **12% effective annual rate**. The task is to pick a single *static* allocation across
five Indian equities — **TCS, HDFCBANK, RELIANCE, SUNPHARMA, ITC** — that maximizes the
probability of clearing the terminal goal.

Two goal schedules are evaluated:

- **Sequence A (early-loaded):** ₹15 L at Y3, ₹25 L at Y7, ₹30 L at Y12.
- **Sequence B (back-loaded):** ₹10 L at Y8, ₹20 L at Y12, ₹40 L at Y16.

The optimum is found two ways: a brute-force search over a discrete weight grid, and a
continuous "bonus" optimization that allows short-selling.

## Notebook

| File | Description |
|------|-------------|
| `Goal_Based_Portfolio_Optimisation.ipynb` | Data, μ/Σ estimation, Monte-Carlo goal engine, discrete grid search, and continuous bonus optimization |

Data: adjusted daily closes 1-Jan-2014 → 29-Dec-2023 (2,465 trading days); μ/Σ from daily
log-returns, annualized (×252) and rescaled to monthly stats.

## Procedure

1. **Estimate inputs.** Pull prices with `yfinance`, compute daily log-returns, annualize to
   `μ, Σ`, and convert to monthly statistics.
2. **Monte-Carlo wealth engine.** On a monthly grid (240 steps): contribute, apply lognormal
   portfolio growth `r_p = exp(z)`, `z ~ N(wᵀμ_m, wᵀΣ_m w)`, accrue debt interest, deduct any
   due goal (borrowing only the gap at the monthly-equivalent 0.949% rate), then sweep any
   positive cash to repay the loan before reinvesting. **Success** means
   `A_240 − D_240 ≥ ₹1.5 Cr`.
3. **Discrete search.** Run 5,000 Monte-Carlo paths (common random numbers, seed 42) for each
   of the 70 valid static portfolios (weights in {0, 0.25, 0.5, 0.75, 1.0} summing to 1), for
   both goal sequences; report the highest success probability.
4. **Bonus — continuous weights.** Repeat with `scipy.optimize.basinhopping` (global) wrapping
   SLSQP (local), allowing short-selling while enforcing `sum(w) = 1`.

## Results

> All numbers below are taken directly from the notebook output cells.

### Per-stock annualized estimates

| Ticker | Ann. log-μ | Ann. σ |
|--------|-----------:|-------:|
| TCS | 15.05% | 24.04% |
| HDFCBANK | 17.49% | 22.46% |
| RELIANCE | 19.62% | 27.77% |
| SUNPHARMA | 8.73% | 29.86% |
| ITC | 11.00% | 25.48% |

Only TCS, HDFCBANK, and RELIANCE clear the 12% borrow rate, so SUNPHARMA and ITC are
unattractive when borrowing is in play.

### Optimal static portfolios

| Sequence | Discrete optimum | P(success) | Continuous (shorts allowed) | P(success) |
|----------|------------------|-----------:|-----------------------------|-----------:|
| A (early-loaded) | 100% RELIANCE | **3.08%** | RELIANCE +1.35, HDFCBANK +0.50, short ITC −0.77 | **22.36%** |
| B (back-loaded) | 50% HDFCBANK + 50% RELIANCE | **82.22%** | HDFCBANK +0.68, RELIANCE +0.70, short ITC −0.42 | **89.42%** |

### Discrete leaderboards (top entries)

**Sequence A:** 100% RELIANCE (3.08%) → 25/75 HDFC/RELIANCE (1.40%) → 25/75 TCS/RELIANCE
(1.04%). Almost every other portfolio clusters near zero.

**Sequence B:** 50/50 HDFC/RELIANCE (82.22%) → 25/75 HDFC/RELIANCE (82.14%) → 100% RELIANCE
(80.84%). The top quartile of portfolios all clear ~70%+.

Other printed values: total nominal contributions ₹71.47 L over 20 years; 70 valid discrete
portfolios; 5,000 paths/portfolio at seed 42.

## Key Takeaways

- **The timing of liabilities reshapes the optimum more than their size.** Sequence A's early
  goals force borrowing before contributions compound, so the optimum collapses to the single
  highest-μ name (100% RELIANCE) and still succeeds only ~3% of the time — the budget is
  structurally tight.
- Sequence B postpones every goal past year 8; the borrow penalty barely fires and the problem
  becomes a clean mean–variance trade-off won by a diversified 50/50 HDFCBANK + RELIANCE blend
  (82%).
- Allowing shorts and leverage (short the low-return ITC/SUNPHARMA, lever RELIANCE) lifts
  success from 3% → 22% (A) and 82% → 89% (B), confirming the same lesson.
