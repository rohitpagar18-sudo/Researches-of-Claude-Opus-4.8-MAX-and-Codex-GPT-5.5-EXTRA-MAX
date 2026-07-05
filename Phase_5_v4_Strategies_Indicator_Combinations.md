# Phase 5 v4 - Tier I Indicator-Combination Strategies

Feed after Phase 4 passes.

## Kickoff prompt

```text
Phases 1-4 passed. Implement Tier I indicator-combination strategies.

Rules:

1. Pre-register every indicator period and threshold.
2. Treat these as weak hypotheses until proven.
3. Compare each to N1/N2/N3/N4 net of all costs.
4. Use correct data: stock volume from cash equity, index volume from futures, OI from futures/options.
5. No look-ahead in pivots, CPR, VWAP, bands, or oscillators.
6. Run Train+Validation smoke reports only.

Do not widen parameters after seeing results.
```

## Tier I stance

Retail indicators are useful only if converted into precise rules and tested net of costs. A failed indicator strategy is a valid result.

Every Tier I result must show:

- Beats buy-and-hold?
- Beats EMA/Supertrend/MACD baselines?
- Survives 2x slippage?
- Trade count sufficient?
- Works on stocks, indices, or both?
- Works intraday, positional, or both?
- Which regime hurts it?

## Strategy list

### I1 - EMA ribbon plus RSI

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Entry: EMAs stacked, price above/below ribbon, RSI confirms regime.
- Exit: ribbon failure, RSI failure, stop, or time-stop.

### I2 - CPR width plus pivot reaction

- Instruments: index futures and stock cash.
- Timeframes: 15m, daily.
- Uses prior completed session CPR only.
- Narrow CPR: breakout mode.
- Wide CPR: mean-reversion mode.
- Trap: gaps can invalidate levels.

### I3 - Volume spike plus price/RSI

- Instruments: stock cash and index futures.
- Timeframes: 15m, daily.
- Entry: RVOL spike, range break, RSI confirmation.
- Exit: stop, target, or volume fade.

### I4 - OI buildup plus price plus moving average

- Instruments: index and stock futures.
- Timeframes: 15m, daily.
- Entry: price/OI quadrant plus MA trend.
- Caveat: OI quadrant is a heuristic, not truth.

### I5 - VWAP plus RSI cross

- Instruments: stock cash and index futures.
- Timeframes: 5m, 15m.
- Entry: VWAP reclaim/loss plus RSI cross.
- Exit: VWAP failure, RSI recross, stop, or session close.

### I6 - Triple confirmation

- Instruments: indices and stocks.
- Timeframes: daily, 15m.
- Components: Supertrend, EMA, RSI.
- Status: falsification target.
- Hypothesis to test: more confirmation reduces false signals.
- Devil's advocate: it often adds lag and reduces trade count.

### I7 - Pivot/SR bounce plus volume

- Instruments: stock cash and index futures.
- Timeframes: 5m, 15m.
- Levels: prior-session pivots only.
- Entry: touch/reject level plus volume confirmation.
- Trap: hindsight level selection is banned.

### I8 - Bollinger/Keltner squeeze

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Variant A: squeeze release breakout.
- Variant B: band mean-reversion.
- Both variants count separately in trial count.
- Breakout variant must use fill haircut.

### I9 - RSI(2) Connors mean-reversion

- Instruments: indices and stocks.
- Timeframe: daily.
- Entry: long-term uptrend plus very low RSI(2).
- Exit: short MA close, RSI recovery, stop, or time-stop.
- Mandatory: mean-reversion spread-crossing penalty and next-bar fills.

### I10 - Oscillator family benchmark

- Instruments: indices and stocks.
- Timeframes: 15m, daily.
- Oscillators: stochastic, CCI, Williams %R.
- Test as a small bounded family, not unlimited combinations.
- Purpose: falsify generic overbought/oversold folklore.

### I11 - MFI/CMF price-volume confirmation

- Instruments: stock cash and index futures.
- Timeframes: 15m, daily.
- Entry: price trigger confirmed by money flow or CMF.
- Exit: flow divergence failure, stop, or time-stop.
- Overlaps Tier F; this is the simple indicator-combo version.

### I12 - Ichimoku/Kijun trend baseline

- Instruments: indices and stocks.
- Timeframes: daily, 60m.
- Entry: price above/below cloud or Kijun/Tenkan confirmation.
- Exit: Kijun break, cloud break, stop, or time-stop.
- Status: weak-proxy baseline unless it beats NULL.

## Parameter discipline

- Small bounded ranges only.
- Each variant counts in the trial penalty.
- No "try every indicator period until it works."
- No divergence rules unless they are fully mechanical.

## Definition of Done

- I1-I12 preregistered.
- Correct data_kind enforced.
- Each runs on Train+Validation for one index and a stock sample.
- Each reports beats-null flags.
- I6 remains a falsification target.
- I8 variants counted separately.
- I9 uses mean-reversion execution penalties.
- No parameter escapes pre-registered bounds.

