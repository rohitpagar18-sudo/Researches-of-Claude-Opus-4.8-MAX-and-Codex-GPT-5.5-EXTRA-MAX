# Phase 0 v4 - Orientation, Feasibility, Scope, and Strategy Governance

Feed this phase first. Do not write strategy code until the feasibility report passes.

## Kickoff prompt

```text
You are an autonomous quantitative research engineer and devil's-advocate reviewer.

Read Phase 0 through Phase 8 v4 completely before coding. Then do only Phase 0:

1. Confirm the runtime: Python version, package manager, disk space, available RAM/CPU, Git status, outbound network, and whether Kite API key/access token are present.
2. Verify live facts before trusting the plan: Kite historical behavior, Kite instrument master, current NIFTY/BANKNIFTY/index lot sizes, current FNO stock universe count, current Zerodha/NSE charges, current expiry rules, and current market holidays.
3. Print a feasibility report by instrument class and timeframe: expected history depth, request count, cache size, likely data-thin cells, and expected wall-clock time.
4. Restate scope: NSE first; stocks plus NIFTY/BANKNIFTY first-class; crypto/FX deferred to Phase 8.
5. Restate the anti-overfit doctrine: locked holdout, preregistration, walk-forward, costs, Deflated Sharpe, SPA/Reality Check, FDR, clustering, and forward validation.

Do not proceed to Phase 1 until the Phase 0 Definition of Done passes. Ask only if a required credential or source is missing.
```

## Goal

Identify a small number of genuinely robust trading strategies across:

- NIFTY and BANKNIFTY.
- Continuous index futures.
- Point-in-time FNO-eligible stocks.
- Intraday and positional timeframes.
- Multiple market regimes: trend, range/chop, high-volatility, low-volatility.

The objective is not to prove that a favorite strategy works. The objective is to falsify weak strategies, rank robust ones, and later forward-validate a small shortlist before any alert or auto-order project begins.

## Scope decisions

### NSE now

This pack targets NSE only:

- NIFTY 50 and BANKNIFTY.
- Index futures and options where data is available.
- Current and historical FNO-eligible stocks with point-in-time membership.
- NSE/FNO costs, taxes, lot sizes, expiry calendars, and corporate actions.

### Stocks are first-class

Individual Indian stocks are not optional. Many strategies are stock-native:

- Cross-sectional momentum.
- Stock-pair stat arb.
- Gap fill / gap-and-go.
- Delivery percentage.
- Accumulation/distribution.
- RVOL and price-volume footprint signals.

The real study should use the full point-in-time FNO universe. Random stocks are only for smoke testing.

### Crypto and Forex later

The methodology transfers to Bitcoin and Forex, but the data and execution models do not. Phase 8 defines the cross-asset extension. Do not mix NSE with crypto/FX in this research run.

## Strategy tiers

Tiers are research categories, not proof of profitability.

| Tier | Meaning |
|---|---|
| NULL | Benchmarks that every strategy must beat. |
| A | Established academic/practitioner edges with strong prior evidence. |
| B | Regime gates, filters, and ML tools. |
| C | Practitioner strategies to test skeptically. |
| D | FII/DII/OI/delivery crowding and flow proxies. |
| E | Novel but rule-based market hypotheses. |
| F | Footprint, accumulation/distribution, and participation features. |
| I | Indicator combinations from retail/books/YouTube. |
| M | Meta/ensemble layer built only after validated sub-strategies exist. |

## Initial strategy roster

### NULL benchmarks

- N1 buy and hold.
- N2 EMA crossover plus ATR filter.
- N3 Supertrend.
- N4 MACD crossover/histogram.

### Tier A

- A1 time-series momentum.
- A2 short-horizon reversal.
- A3 cointegration pairs: index pair and stock pairs.
- A4 VIX-conditioned regime gate.
- A5a volatility risk premium signal.
- A5b options-selling implementation, forward-test-only.
- A6 Donchian/Turtle breakout.
- A7 cross-sectional momentum / relative strength.
- A8 residual or sector-neutral momentum for stocks.

### Tier B

- B1 HMM regime gate.
- B2 Hurst trend/reversion gate.
- B3 Kalman trend filter.
- B4 gradient-boosted next-bar classifier with purge/embargo.
- B5 ADX/DMI trend-strength gate.
- B6 market breadth / sector participation gate.

### Tier C

- C1 volume profile / POC reversion.
- C2 anchored VWAP reaction.
- C3 mechanical SMC/order-block/liquidity-sweep falsification target.
- C4 opening range breakout.
- C5 gap fill / gap-and-go.
- C6 failed breakout / liquidity sweep reversal.
- C7 prior-day high/low and opening auction acceptance/rejection.

### Tier D

- D1 FII index-futures positioning extreme, index only.
- D2 FII cash plus futures convergence gate, index only.
- D3 OI buildup/unwind feature, index and stock futures.
- D4 delivery percentage spike filter, stocks; index proxy only.

### Tier E

- E1 futures basis / roll-yield signal.
- E2 overnight vs intraday return split conditioned by FII cash.
- E3 dispersion regime gate.
- E4 scheduled-event drift: RBI, Budget, expiry, earnings.

### Tier F

- F1 OBV/ADL/CMF accumulation-distribution divergence.
- F2 relative volume plus range expansion.
- F3 volume contraction then expansion breakout.
- F4 delivery/volume absorption filter.
- F5 live order-book/depth imbalance, forward-test-only unless recorded.

### Tier I

- I1 EMA ribbon plus RSI.
- I2 CPR width plus pivot reaction.
- I3 volume spike plus price/RSI confirmation.
- I4 OI buildup plus price plus moving average.
- I5 VWAP plus RSI cross.
- I6 Supertrend plus EMA plus RSI triple-confirmation falsification target.
- I7 pivot/SR bounce plus volume.
- I8 Bollinger/Keltner squeeze breakout plus band mean-reversion variant.
- I9 RSI(2) Connors mean-reversion.
- I10 oscillator family benchmark: stochastic, CCI, Williams %R.
- I11 MFI/CMF price-volume confirmation.
- I12 Ichimoku/Kijun trend baseline.

### Tier M

- M1 regime-aware ensemble from validated sub-strategies only.

## Promotion, demotion, and removal policy

Performance state is separate from research tier:

- Candidate: eligible for full sweep.
- Validated: passes Phase 6 holdout and statistical gates.
- Forward-watch: historically valid but needs Phase 7.
- Retired: failed null, cost, holdout, FDR, SPA, or robustness checks.
- Forward-only: cannot be historically ranked.
- Invalid: look-ahead, holdout leakage, wrong data source, survivorship, or unfillable execution.

No strategy is deleted silently. Retired and invalid strategies stay in reports with reasons.

## Risk doctrine

The engine must enforce:

- Start equity shown as INR 1,00,000 for interpretability.
- Default risk per trade <= 1% of equity; never above 2%.
- Stop-loss defined before entry.
- Position size derived from risk, stop distance, lot size, margin, and liquidity.
- Max trades per day for intraday strategies, default 2 or 3 unless pre-registered.
- Max daily loss and max weekly loss hard stops.
- No averaging down unless a strategy explicitly pre-registers pyramiding rules.
- Reward:risk is reported, not blindly forced. Mean-reversion can have reward:risk below 1 if expectancy is positive, but it must survive costs and stress tests.

Every trade log must answer:

- What is my risk?
- What if I am wrong?
- Where is the stop?
- Is this strategy or gambling?
- Am I overtrading?
- Does expectancy justify the reward:risk?

## Sources to verify at runtime

- Kite historical docs: https://kite.trade/docs/connect/v3/historical/
- Kite instrument master docs: https://kite.trade/docs/connect/v3/market-quotes/
- Zerodha charges: https://zerodha.com/charges/
- NSE derivatives underlyings: https://www.nseindia.com/products-services/equity-derivatives-list-underlyings-information
- NSE circulars and market holidays from NSE's current site.

## Definition of Done

- Environment and credentials checked.
- Live fact verification printed with drift notes.
- FNO universe count verified live, not hard-coded.
- Feasibility matrix printed for index, futures, options, stocks, and timeframes.
- Request count, cache size, and expected fetch time estimated.
- Scope and anti-overfit doctrine restated.
- No strategy code written yet.

