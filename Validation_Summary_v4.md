# Validation Summary - NSE Strategy Research Pack v4

Date: 2026-06-24

This is a devil's-advocate validation of the supplied Phase 0-7 documents. The original pack is strong and should not be discarded. Version 4 keeps its core architecture, but tightens the weak spots before any backtest is run.

## Executive verdict

Use the regenerated v4 phase pack instead of executing the original documents directly.

The original documents already cover the hardest parts: point-in-time FNO stock membership, anti-overfitting, locked holdout, realistic costs, full instrument x strategy x timeframe sweeps, and a paper-trading bridge. The main gaps were:

1. Some "verified current" facts were too hard-coded for a live market system. v4 requires runtime verification and API probes before execution.
2. The FNO stock universe must be all current and historical eligible stocks by default. Random stocks are allowed only for smoke tests, not research conclusions.
3. Strategy tiers needed explicit promotion, demotion, and retirement rules based on performance.
4. Money-footprint logic needed to be stronger and more mechanical: OBV, CMF, ADL, RVOL, delivery, absorption, volume contraction, failed breakout, and order-book/depth items.
5. Risk controls needed to become machine-enforced, not just described: risk per trade, stop before entry, max daily loss, max trades per day, lot-size constraints, and risk-of-ruin checks.
6. The research loop needed a stricter "AI scientist" fence so Claude/Codex can propose new strategies without mining the locked holdout.
7. Cross-asset support for Bitcoin and Forex needed to be separated into a later adapter phase, not mixed into the NSE study.

## Direct answers

### Can strategies be applied to individual stocks?

Yes. The v4 pack treats individual FNO-eligible Indian stocks as first-class instruments, not a side experiment. Use the full point-in-time FNO universe unless compute is impossible. Some strategies are expected to work better on stocks than indices:

- Cross-sectional momentum and relative strength.
- Stock-pair stat arb.
- Gap fill / gap-and-go.
- Delivery and accumulation/distribution filters.
- RVOL, OBV, CMF, ADL, and price-volume footprint strategies.

Index-specific strategies still matter, especially NIFTY/BANKNIFTY futures, VIX-conditioned regimes, FII index positioning, futures basis, and expiry behavior.

### Can the same system test Bitcoin or Forex?

The methodology can transfer; the data layer cannot. NSE, crypto, and FX have different hours, cost models, instruments, funding, slippage, and microstructure. Version 4 adds Phase 8 as a clean cross-asset extension plan. Do not mix NSE, crypto, and FX in one discovery tournament; each market needs its own adapter, cost model, and multiple-testing count.

### Will YouTube/book-style indicators be tested?

Yes. Version 4 keeps Tier I and expands it. RSI, VWAP, volume, OI, CPR, pivots, SMA/EMA/WMA, MACD, Bollinger/Keltner squeeze, RSI(2), MFI, CMF, stochastic/CCI/Williams %R, and Ichimoku/Kijun-style logic can be tested systematically. They are treated as weak hypotheses until they beat null baselines net of costs.

### Should we use all 180/220 FNO stocks or random stocks?

Use all point-in-time FNO-eligible stocks for the real study. If compute is limited:

- Smoke test: a small random set is allowed only to verify code.
- Research subset: use a pre-registered stratified sample by sector, liquidity, and volatility.
- Final ranking: full universe is preferred.

Hand-picking "good stocks" before testing is selection bias.

## Strategy tier changes in v4

### Keep as core candidates

- A1 time-series momentum.
- A3 cointegration pairs, expanded to stock pairs.
- A6 Donchian/Turtle breakout.
- A7 cross-sectional momentum / relative strength.
- A8 residual or sector-neutral momentum variant.
- C4 opening range breakout.
- C5 gap fill / gap-and-go.
- E1 futures basis / roll yield.
- M1 regime-aware ensemble.

### Keep but demote to baseline or weak hypothesis

- EMA crossover, Supertrend, and MACD are NULL baselines, not "discoveries".
- SMC/order-block language is a falsification target unless it becomes fully mechanical.
- Triple-confirmation indicator stacks are a falsification target.
- OI buildup and delivery percentage are features/filters unless they prove standalone value.

### Add in v4

- B6 market breadth / sector participation gate.
- C6 failed breakout / liquidity sweep reversal, defined mechanically.
- C7 previous-day high/low and opening auction acceptance/rejection.
- F1 accumulation/distribution divergence using OBV, ADL, CMF.
- F2 relative-volume plus range-expansion breakout.
- F3 volume-contraction pattern into expansion breakout.
- F4 delivery/volume absorption filter for stocks.
- F5 live depth/order-book imbalance, forward-test-only unless depth is recorded.
- I10 oscillator family benchmark: stochastic, CCI, Williams %R.
- I11 MFI/CMF price-volume confirmation.
- I12 Ichimoku/Kijun trend baseline.

## Promotion and removal rules

Strategies are not truly promoted or removed until Phase 6/7 results exist.

Promote to "validated candidate" only if it passes:

- Beats N1 buy-and-hold and relevant NULL baselines net of all costs.
- Positive holdout OOS risk-adjusted performance.
- Deflated Sharpe clears the pre-registered threshold.
- Calmar and drawdown are acceptable for the strategy class.
- SPA / Reality Check does not reject the edge as data snooping.
- FDR survives for stock-panel discoveries.
- Cost sensitivity survives at 2x slippage.
- Not data-thin, or explicitly labeled exploratory.
- Forward validation in Phase 7 does not collapse.

Retire or demote if:

- It only works before costs.
- It fails holdout after good Train/Validation.
- It is too sensitive to small parameter changes.
- It fails under 2x slippage.
- It works only on a tiny correlated cluster without FDR support.
- It needs discretionary/hindsight definitions.

Retired strategies should remain in the dashboard as an audit trail, not disappear.

## Public sources checked

- Kite historical candles and continuous futures behavior: https://kite.trade/docs/connect/v3/historical/
- Kite instrument list behavior and token reuse warning: https://kite.trade/docs/connect/v3/market-quotes/
- Zerodha brokerage and statutory charges: https://zerodha.com/charges/
- NSE equity derivatives underlyings page: https://www.nseindia.com/products-services/equity-derivatives-list-underlyings-information
- Public reporting on NSE/BSE expiry-day change from September 2025: https://economictimes.indiatimes.com/markets/stocks/news/nse-fo-expiry-shifts-to-tuesday-bses-to-thursday-from-sept-1/articleshow/121919549.cms

## Regenerated files

- Phase_0_v4_Orientation_Feasibility_Strategy_Governance.md
- Phase_1_v4_Data_Pipeline_PointInTime_Costs.md
- Phase_2_v4_Methodology_Engine_Risk_Validation.md
- Phase_3_v4_Strategies_Core_NULL_A_B.md
- Phase_4_v4_Strategies_Practitioner_Flow_Novel.md
- Phase_5_v4_Strategies_Indicator_Combinations.md
- Phase_6_v4_Full_Run_Dashboard_Meta_Discovery.md
- Phase_7_v4_Paper_Trading_Forward_Validation.md
- Phase_8_v4_Cross_Asset_Extension_Alert_Readiness.md

