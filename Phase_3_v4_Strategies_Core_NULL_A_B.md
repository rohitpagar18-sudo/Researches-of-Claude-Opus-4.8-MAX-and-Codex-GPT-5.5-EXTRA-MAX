# Phase 3 v4 - Core Strategies: NULL, Tier A, and Tier B

Feed after Phase 2 passes.

## Kickoff prompt

```text
Phase 2 passed. Implement NULL benchmarks, Tier A strategies, and Tier B gates.

For each strategy or gate:

1. Add preregistration entry first.
2. Implement as StrategyBase subclass or gate module.
3. Define exact entry, exit, stop, target/time-stop, and risk assumptions.
4. Declare data_kind and allowed timeframes.
5. Run on Train+Validation only for one index and a small stock sample.
6. Confirm finite metrics and no guardrail violations.

Do not run the full sweep. That belongs to Phase 6.
```

## NULL benchmarks

NULL strategies are not discoveries. They are the bar to beat.

### N1 - Buy and hold

- Instruments: indices and stocks.
- Timeframes: daily.
- Purpose: benchmark.
- Failure flag for active strategies: does not beat N1 net of costs.

### N2 - EMA crossover plus ATR filter

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Params: fast EMA, slow EMA, ATR filter.
- Status: baseline.

### N3 - Supertrend

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Params: ATR period, multiplier.
- Status: baseline.

### N4 - MACD

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Params: fast, slow, signal.
- Status: baseline.

## Tier A strategies

### A1 - Time-series momentum

- Type: positional.
- Instruments: indices and stocks.
- Timeframes: daily, weekly.
- Hypothesis: trailing return direction persists.
- Entry: next open after momentum state turns positive/negative.
- Exit: state flip, ATR stop, or time-stop.
- Gates: B1/B2/B5/B6 optional.
- Main traps: trend crash, crowded unwind, look-ahead in trailing return.

### A2 - Short-horizon reversal

- Type: intraday/positional.
- Instruments: indices and stocks.
- Timeframes: 15m, 60m, daily.
- Hypothesis: extreme short-term moves partially revert.
- Entry: normalized move exceeds z-score or ATR threshold.
- Exit: mean reversion, stop, or time-stop.
- Mandatory: spread-crossing penalty.
- Main traps: fading trend days, cost drag.

### A3 - Cointegration pairs

- Type: stat arb.
- Instruments: NIFTY/BANKNIFTY pair and stock pairs.
- Timeframes: 15m, 60m, daily.
- Hypothesis: cointegrated spreads mean-revert while cointegration holds.
- Stock pairs: prefer same sector and overlapping eligibility windows.
- Entry: spread z-score beyond threshold.
- Exit: z-score mean reverts, stop, time-stop, or cointegration breaks.
- Trial count: every tested pair counts.

### A4 - VIX-conditioned regime gate

- Type: gate.
- Instruments: index-led regime, can inform stocks.
- Timeframe: daily.
- Uses India VIX plus realized volatility.
- Output: low/normal/high vol state.

### A5a - Volatility risk premium signal

- Type: positional signal/gate.
- Instruments: index.
- Timeframe: daily.
- Uses implied vol proxy minus realized vol.
- Options-selling implementation is not backtest-ranked here.

### A5b - Defined-risk options premium strategy

- Type: forward-test-only.
- Instruments: index options.
- Requires recorded option-chain data.
- Never ranked against historical strategies until enough forward data exists.

### A6 - Donchian / Turtle breakout

- Type: positional trend.
- Instruments: indices and stocks.
- Timeframes: daily, weekly.
- Entry: breakout above N-day high or below N-day low.
- Exit: opposite M-day channel, ATR trailing stop, or time-stop.
- Gates: B2/B5 recommended.
- Main traps: whipsaw in ranges, breakout fill optimism.

### A7 - Cross-sectional momentum / relative strength

- Type: stock-only positional.
- Instruments: FNO stock universe.
- Timeframes: daily, weekly.
- Entry: rank eligible stocks by trailing momentum, optionally skip latest month.
- Portfolio: long top bucket, optional short bottom bucket if feasible; long-only variant required.
- Exit: rebalance or risk stop.
- Mandatory: point-in-time membership, turnover cost, FDR.

### A8 - Residual / sector-neutral momentum

- Type: stock-only positional.
- Instruments: FNO stock universe.
- Timeframes: daily, weekly.
- Hypothesis: stock-specific momentum can be cleaner than raw sector beta.
- Method: rank residual returns after sector/index beta adjustment.
- Purpose: reduce "all banks moved together" false discoveries.
- Mandatory: sector map and clustering.

## Tier B gates and tools

### B1 - HMM regime gate

- Fit only within walk-forward training windows.
- Handle label switching.
- Output bull/bear/chop probabilities.

### B2 - Hurst regime gate

- Detect trend vs mean-reversion tendency.
- Weak alone; used as a gate.

### B3 - Kalman trend filter

- Low-lag trend estimate.
- Feature/gate, not standalone unless separately validated.

### B4 - Gradient-boosted classifier

- Type: ML.
- Instruments: indices and stocks.
- Timeframes: 5m, 15m.
- Features: lagged returns, volatility, time of day, volume/OI where valid, gates.
- Mandatory: purge/embargo, no leakage, probability calibration, feature importance stability.
- Excluded if it cannot beat N2/N3/N4 OOS.

### B5 - ADX/DMI trend-strength gate

- Gate trend/breakout systems.
- Useful for avoiding chop.
- Weak as standalone.

### B6 - Market breadth / sector participation gate

- Stocks and index regime.
- Inputs: advance/decline from panel, percent above moving average, sector breadth, dispersion.
- Use: confirm broad participation behind trend signals.
- Devil's advocate: breadth computed from today's universe on old dates is survivorship; must use point-in-time eligible panel.

## Definition of Done

- N1-N4 implemented as baselines.
- A1-A8 preregistered and runnable on Train+Validation.
- B1-B6 gates implemented and reusable.
- A5b marked forward-test-only.
- A7/A8 use point-in-time stock panel.
- Pair scans count all tested pairs.
- B4 uses purge/embargo.
- Reports include metrics, INR 1,00,000 block, regime buckets, and failure modes.

