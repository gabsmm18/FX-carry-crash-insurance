# FX Carry with a Momentum Filter: Crash Insurance or Alpha?

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/gabsmm18/fx-carry-momentum/blob/main/notebooks/carry_momentum_fx.ipynb)
![Monthly signal](https://github.com/gabsmm18/fx-carry-momentum/actions/workflows/monthly_signal.yml/badge.svg)

> **TL;DR.** On G10 currencies (2000–2026, net of costs), a 3-month momentum filter works as **crash insurance, not as alpha**. It nearly eliminates the carry trade's negative skewness (**−0.81 → −0.10**) and turns the worst crisis months positive (**Oct-2008: −11.2% → +3.6%**), but it **halves the Sharpe ratio (0.40 → 0.19)** through whipsaw losses and 5× higher turnover. The result holds in 10 of 12 parameter combinations.

![Cumulative returns and drawdowns](figures/cumulative_drawdown.png)

---

## Motivation

Uncovered Interest Parity says that high-yielding currencies should depreciate by exactly their interest advantage, so borrowing in low-yield currencies to invest in high-yield ones should earn nothing. Empirically it does not hold (the **forward premium puzzle**, Fama 1984), so the **carry trade** earns a positive average return.

The standard explanation is a **crash-risk premium**: carry earns small, steady gains and suffers rare, violent losses when crowded positions unwind (Brunnermeier, Nagel & Pedersen 2008; Menkhoff et al. 2012). Examples are 2008, the 2015 SNB de-peg and the August 2024 yen unwind.

**Research question:** can a simple momentum filter, which drops a carry position when its recent trend turns against it, avoid these crashes without giving up the carry premium?

## Strategy

| Step | Rule |
|---|---|
| Universe | EUR, GBP, JPY, CHF, AUD, NZD, CAD, NOK, SEK vs USD |
| Carry signal | 3-month interest differential vs USD |
| Carry portfolio | Long the 3 highest-yielders, short the 3 lowest (±1/3 each), dollar-neutral, rebalanced monthly |
| Momentum filter | Close a long (short) if its 3-month excess return is negative (positive) |
| Excess return | `xr = Δln S + (i* − i)/12`, the return of a rolled 1-month forward under CIP |
| Costs | 3 bp per unit of turnover |
| Benchmarks | Pure carry; pure time-series momentum (long if 3M return > 0, short otherwise) |

Signals use only information available at month-end *t* and are applied to returns in month *t+1* (no look-ahead). Parameters (3 currencies per leg, 3-month lookback) were fixed **before** looking at results.

## Results

**Full sample, Jan 2000 – Oct 2026, raw exposure, net of costs**

| | Carry | TS Momentum | Carry + Filter |
|---|---:|---:|---:|
| Annual return | 2.8% | 0.2% | 1.4% |
| Annual volatility | 7.0% | 6.2% | 7.2% |
| **Sharpe ratio** | **0.40** | 0.04 | 0.19 |
| **Skewness** | **−0.81** | +0.54 | −0.10 |
| Worst month | −11.2% | −5.8% | −8.0% |
| Max drawdown | −33% | −40% | −37% |
| Hit rate | 58% | 48% | 52% |
| Avg. monthly turnover | 0.10 | 0.52 | 0.55 |

**Crisis months: the filter does what it was designed to do**

| Month | Event | Carry | Carry + Filter |
|---|---|---:|---:|
| Oct 2008 | Lehman / global deleveraging | −11.2% | **+3.6%** |
| Jan 2015 | SNB removes EUR/CHF floor | −3.5% | −0.7% |
| Mar 2020 | Covid dash for dollars | −5.7% | **+0.9%** |
| Sep 2022 | Strong-dollar shock | −4.6% | **+2.9%** |

Ahead of Oct 2008 the filter had already halved gross exposure (2.0 → 1.0), because high-yielders had been losing for months.

**Sub-samples: Sharpe ratio**

| Period | Carry | TS Momentum | Carry + Filter |
|---|---:|---:|---:|
| 2000–2007 | **0.95** | 0.25 | 0.54 |
| 2008–2015 | 0.03 | **0.43** | 0.37 |
| 2016–2026 | **0.42** | −0.50 | −0.27 |

**Robustness.** Across 2/3/4 currencies per leg and 1/3/6/12-month lookbacks, the filter lowers the Sharpe ratio in **10 of 12** combinations.

## Interpretation

1. **Carry behaves as theory predicts:** a positive premium paired with strongly negative skewness, i.e. compensation for crash risk.
2. **The filter is insurance with a negative carry of its own.** It avoids the fast crashes but bleeds in trendless markets: it exits, the currency rebounds, it re-enters late (*whipsaw*). As the drawdown chart shows, carry's worst drawdown is one sharp hole in 2008, while the filter's is a slow decline from 2015 onwards.
3. **The filter inherits momentum's fate.** It helps only in 2008–2015, the regime of repeated crises, and loses after 2016, when FX time-series momentum itself performed poorly in this sample.
4. **Implication:** momentum earns exactly when carry crashes (+9.6% in Oct 2008). This suggests **combining** the two strategies as diversifiers (à la Asness, Moskowitz & Pedersen 2013) rather than using one to switch the other off. This is the next test.

## Limitations

- **Rates data:** OECD 3-month interbank rates (monthly averages) proxy for forward-implied rates; the cross-currency basis is ignored. The EUR and GBP series on FRED end in Jan 2026, so these currencies drop out of the most recent months.
- **Small universe:** 9 currencies with 3 per leg is concentrated and noisy. Academic studies use 30–50 currencies, including emerging markets. The USD is excluded from the cross-section.
- **No formal significance test:** a Sharpe of 0.4 over 26 years has a standard error of roughly 0.2.
- **Implementation:** retail brokers pay less than the theoretical carry. 3 bp costs are an institutional assumption.

## Repository

```
notebooks/carry_momentum_fx.ipynb     research notebook: theory, backtest, crises, robustness
src/strategy.py                       data download (FRED API), signals, backtest, metrics
src/live_signal.py                    monthly signal + optional IBKR paper-account rebalancing
.github/workflows/monthly_signal.yml  computes and commits the signal on the 8th of each month
signals/                              latest.json and history.csv (written by the workflow)
figures/                              charts used in this README
```

## How to run

1. Get a free FRED API key: [fredaccount.stlouisfed.org/apikeys](https://fredaccount.stlouisfed.org/apikeys).
2. **Colab:** click the badge, add the key as a Colab Secret named `FRED_API_KEY`, then `Runtime → Run all`.
3. **Locally:**
   ```bash
   pip install -r requirements.txt
   export FRED_API_KEY=your_key
   python src/live_signal.py                               # signal only
   python src/live_signal.py --broker ib --capital 100000  # dry run vs IBKR paper account
   ```
4. **Automated signal:** add `FRED_API_KEY` as a repository secret (*Settings → Secrets and variables → Actions*).

Never commit your API key.

## References

- Fama, E. (1984). Forward and spot exchange rates. *Journal of Monetary Economics*.
- Brunnermeier, M., Nagel, S. & Pedersen, L. (2008). Carry trades and currency crashes. *NBER Macroeconomics Annual*.
- Lustig, H., Roussanov, N. & Verdelhan, A. (2011). Common risk factors in currency markets. *Review of Financial Studies*.
- Menkhoff, L., Sarno, L., Schmeling, M. & Schrimpf, A. (2012). Carry trades and global FX volatility. *Journal of Finance*.
- Menkhoff, L., Sarno, L., Schmeling, M. & Schrimpf, A. (2012). Currency momentum strategies. *Journal of Financial Economics*.
- Asness, C., Moskowitz, T. & Pedersen, L. (2013). Value and momentum everywhere. *Journal of Finance*.

---

**Author:** Mattia Mariani, MSc Applied Economics and Finance, Copenhagen Business School

*Educational and research project. Not investment advice.*
