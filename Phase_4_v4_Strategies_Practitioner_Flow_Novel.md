# Phase 4 v4 - Practitioner, Flow, Novel, and Footprint Strategies

Feed after Phase 3 passes.

## Kickoff prompt

```text
Phase 3 passed. Implement Tier C, D, E, and F strategies/features.

For each:

1. Pre-register hypothesis, data, timeframes, bounds, and failure modes.
2. Implement exact mechanical rules.
3. Respect labels: standalone, gate, feature, filter, falsification target, or forward-test-only.
4. Use correct data_kind: index volume uses futures; stock volume uses cash equity; OI uses futures/options.
5. Run Train+Validation smoke reports only.

Do not promote a feature into a ranked standalone strategy unless pre-registered as such.
```

## Tier C - Practitioner strategies

### C1 - Volume profile / POC reversion

- Instruments: index futures and stock cash.
- Timeframes: 5m, 15m, 60m.
- Entry: price extends away from prior completed session POC and reversion trigger fires.
- Exit: POC touch, stop, or time-stop.
- Trap: computing POC with current/future session bars.

### C2 - Anchored VWAP reaction

- Instruments: index futures and stock cash.
- Timeframes: 15m, 60m, daily.
- Anchor rules must be pre-registered: session open, event date, swing high/low defined without future bars.
- Entry: reclaim/reject AVWAP with confirmation.
- Exit: stop, band target, or time-stop.

### C3 - Mechanical SMC/order-block/liquidity-sweep

- Status: falsification target.
- Instruments: indices and stocks.
- Timeframes: 5m, 15m.
- Only allowed if every zone and sweep is defined without discretion.
- Do not widen rules to rescue it after poor results.

### C4 - Opening range breakout

- Instruments: index futures and stock cash.
- Timeframes: 5m, 15m.
- Entry: break opening range after 15/30/45 minutes with volume or volatility confirmation.
- Exit: opposite range stop, R target, trailing stop, or session close.
- Mandatory: breakout missed-fill haircut.
- Gates: A4/B2/B5/B6.

### C5 - Gap fill / gap-and-go

- Instruments: stocks primarily; index optional but expected weaker.
- Timeframes: daily plus 5m/15m open.
- Entry:
  - small low-volume gap: fade toward prior close.
  - large high-RVOL gap: follow on pullback or ORB.
- Exit: prior close, R target, stop, or session close.
- Event gaps flagged separately.

### C6 - Failed breakout / liquidity sweep reversal

- Instruments: index futures and stock cash.
- Timeframes: 5m, 15m, 60m.
- Hypothesis: stop-run beyond prior high/low that fails back inside can reverse.
- Entry: price breaks a known level by a threshold, closes back inside, and volume/range confirms failure.
- Exit: midrange target, opposite liquidity level, stop, or time-stop.
- Trap: hindsight-chosen levels. Only prior session high/low, opening range, CPR/pivots, and rolling highs/lows are allowed.

### C7 - Prior-day high/low and opening auction acceptance

- Instruments: index futures and stock cash.
- Timeframes: 5m, 15m.
- Entry: acceptance above prior high/low or rejection back inside prior range.
- Exit: next reference level, stop, or session close.
- Purpose: test common market-structure levels mechanically.

## Tier D - Crowding and flow proxies

Public FII/DII data is aggregate and end-of-day. It does not prove intent. Treat it as crowding or confirmation, not mind-reading.

### D1 - FII index-futures positioning extreme

- Instruments: index only.
- Timeframe: daily.
- Signal: rolling percentile of net positioning.
- Default interpretation: contrarian at extremes.
- Exit: percentile normalizes, stop, or time-stop.

### D2 - FII cash plus futures convergence

- Type: gate.
- Instruments: index only.
- Use only to confirm or veto other index signals.

### D3 - OI buildup/unwind

- Type: feature, not standalone by default.
- Instruments: index and stock futures.
- Caveat: OI grid cannot identify aggressor, hedge leg, or arbitrage.

### D4 - Delivery percentage spike

- Type: stock filter.
- Instruments: stocks.
- Use with price/volume context.
- Caveat: delivery can reflect passive flows, block deals, or settlement behavior, not only conviction.

## Tier E - Novel hypotheses

### E1 - Futures basis / roll yield

- Instruments: index and stock futures.
- Timeframe: daily.
- Use unadjusted futures plus spot and days-to-expiry.
- Handle near-expiry basis convergence.

### E2 - Overnight vs intraday return split

- Instruments: index.
- Timeframes: daily decomposition.
- Condition on FII cash as-of values.
- Watch for revised data leakage.

### E3 - Dispersion regime

- Type: gate.
- Instruments: index and stock universe.
- Use sector-index dispersion and/or point-in-time stock panel dispersion.
- High dispersion may favor stock selection; low dispersion may favor index trend.

### E4 - Scheduled-event drift

- Events: RBI policy, Union Budget, expiry, major index rebalances, stock earnings.
- Dates must be scheduled/known in advance.
- Sample sizes will be small and must be flagged.

## Tier F - Footprint and accumulation/distribution

These are the user's "real money footprint" layer. They convert capital-flow ideas into mechanical, testable features.

### F1 - OBV/ADL/CMF accumulation-distribution divergence

- Instruments: stock cash and index futures.
- Timeframes: 15m, daily.
- Hypothesis: participation is improving when price is flat/down but OBV, ADL, or CMF rises.
- Entry: divergence plus price trigger.
- Exit: divergence failure, stop, or time-stop.
- Caveat: indicators are proxies, not proof of institutional buying.

### F2 - Relative volume plus range expansion

- Instruments: stock cash and index futures.
- Timeframes: 5m, 15m, daily.
- Entry: RVOL above threshold plus range expansion plus close near high/low.
- Exit: trailing stop, VWAP loss, or time-stop.
- Purpose: separate real participation from low-volume drift.

### F3 - Volume contraction then expansion breakout

- Instruments: stocks primarily.
- Timeframes: daily, 60m.
- Hypothesis: volatility/volume contraction followed by expansion can mark accumulation breakout.
- Entry: contraction regime ends with breakout and RVOL confirmation.
- Exit: failed breakout, ATR stop, or time-stop.
- This is a mechanical version of VCP/Wyckoff-style ideas.

### F4 - Delivery/volume absorption filter

- Instruments: stocks.
- Timeframe: daily.
- Use: filter longs when price holds a range despite high volume/delivery, suggesting absorption.
- Must define absorption mechanically: range width, close location, volume percentile, delivery percentile.

### F5 - Live order-book/depth imbalance

- Status: forward-test-only unless depth has been recorded.
- Instruments: liquid futures/stocks.
- Data: market depth snapshots from recorder.
- Use: feature for fill quality and short-horizon pressure.
- Caveat: displayed liquidity can vanish; spoofing-like behavior makes naive depth signals fragile.

## Definition of Done

- C1-C7, D1-D4, E1-E4, F1-F5 preregistered.
- Correct data_kind enforced.
- Feature-only items cannot rank standalone unless explicitly pre-registered.
- Footprint features generate observable signals without future bars.
- Each runnable item produces finite Train+Validation reports.
- Result pages show fact / assumption / unknown for D/E/F strategies.

