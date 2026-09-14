# Capstone project plan

Personal working plan for the Stock Markets Analytics Zoomcamp 2026 capstone.
The short version submitted as the homework answer is in `homework1.ipynb`.

**Goal.** Predict which IBEX 35 constituents will outperform the equal-weighted average of the index members over the following 30 trading days, and test whether Spanish and euro-area interest-rate conditions add predictive power beyond price-based features, particularly for banks, which dominate the Spanish market.

**Universe and period.** IBEX 35 members, 2010-2026, using point-in-time membership reconstructed from BME's *Comite Asesor Tecnico* announcements to avoid survivorship bias. Constituents are checked at each rebalance, and a stock leaving the index is exited at the next rebalance date. Unicaja is included only from its June 2017 listing.

**Data.**
* *Prices*: daily dividend-adjusted closes and volumes from Yahoo Finance (`yfinance`, `auto_adjust=True`) for all constituents.
* *Market context*: IBEX 35, Euro Stoxx 50, S&P 500, VIX, EUR/USD and Brent, daily.
* *Rates*: ECB deposit rate, 3-month Euribor, and Spanish and German 10-year yields from the ECB Data Portal (daily), from which I derive the Spain-Germany spread and the 10y-3m slope. These are lagged one day only: they are market data known the same day, unlike macro statistics such as CPI or GDP.
* *Sector*: an ICB/GICS label per stock.

**Features.** RSI(14), MACD(12,26,9), price relative to its 50- and 200-day moving averages, returns over 1/7/30/90/252 days, 30-day realised volatility, volume relative to its 20-day average, plus the rate block and the sector label. The indicators are implemented directly in pandas: TA-Lib needs a C toolchain on Windows and `pandas-ta` conflicts with the installed numpy.

**Target.** Binary: whether the 30-trading-day forward return is above the equal-weighted mean of that period's constituents. Roughly balanced by construction.

**Validation.** Walk-forward with expanding windows and a 30-day embargo between train and test, so the overlapping label windows cannot leak. Train 2010-2019, validate 2020-2022, and hold out 2023-2026 until the end. Models: logistic regression as a baseline, decision tree and random forest, each run with and without the rate block. Reference baseline: a 12-month momentum ranking on its own.

**Metrics.** AUC and rank correlation for the prediction; annualised return, Sharpe ratio, maximum drawdown and hit rate for the strategy.

**Strategy and costs.** Long the 5 highest-ranked stocks, rebalanced every 30 trading days. Costs include broker commission, an assumed bid-ask spread scaled to each stock's liquidity, and Spain's 0.2% financial transaction tax applied to purchases only, from 16 January 2021 onward, and only for issuers above 1 billion EUR of market capitalisation.

**Benchmarks.** Equal-weight buy-and-hold of the same universe with dividends reinvested, and the IBEX 35 **total return** index (*IBEX 35 con dividendos*) rather than the Yahoo price index `^IBEX`, which excludes the 3-4% annual dividend yield and would flatter the strategy.

**Known limitations.** About 200 non-overlapping periods across 35 highly correlated stocks is a small effective sample for the number of features, so results are reported with that caveat and the six-bank subgroup is treated as exploratory rather than conclusive.
