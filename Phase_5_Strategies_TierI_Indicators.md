# PHASE 5 — Tier I: Indicator-Combination Strategies (v3)

> Feed after Phase 4 passes. These are the popular retail combos (RSI/EMA/WMA/Volume/OI/CPR/Pivot/Bollinger/MACD). Tested honestly — included to be **falsified** as much as confirmed.

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phases 1-4 are complete and passed. Implement Tier I (indicator-combination strategies) per this Phase 5
doc, on BOTH the index instruments and the F&O stock universe (point-in-time), including the v3 additions
I8 (Bollinger squeeze->breakout) and I9 (RSI(2) Connors mean-reversion). For each: pre-register hypothesis +
structural rationale + bounded ranges, subclass StrategyBase, set instruments and data_kind correctly
(index volume->futures; stock volume->cash_equity; OI logic->stock futures), wire any consumed regime gates
(A4/B2/B5/E3). These combos are popular but weakly evidenced: each MUST be compared against the NULL
baselines (buy-hold, EMA cross, Supertrend, MACD) and flagged "no edge over baseline" if it fails net of
costs. Do not curve-fit indicator periods outside pre-registered bounds. Run each once on Train+Val
(one index + a few stocks) to confirm a finite report incl. per-regime buckets. Full sweep is Phase 6.
Verify Definition of Done.
```

---

## 1. Objective & honest stance

Indicator combinations are the most common retail strategies (YouTube/books) and the **most overfit-prone, least-evidenced** category. We test them so the dashboard can show, with data, which (if any) survive realistic costs, realistic fills, and the multiple-testing penalty — and which are noise dressed as signal. **A Tier I strategy that fails to beat buy-and-hold net of costs is a valid, reportable result.** Do not rescue it by widening the parameter search.

## 2. Shared rules for Tier I

- **Beat the null.** Every Tier I result carries a "beats N1/N2/N3/N4 net of costs?" flag.
- **No look-ahead in indicators.** Pivots/CPR use the *prior* completed session only; RSI/EMA/WMA/MACD/Bollinger use only closed bars; volume averages exclude the current bar.
- **Instruments:** run on both index (volume→futures) and F&O stocks (volume→cash_equity). OI-based (I4) needs stock futures / stock OI.
- **Execution realism is mandatory** — especially the spread-crossing penalty on the mean-reversion combos (I9 in particular) and the breakout fill-haircut on breakout combos (I8).
- **Bounded periods, pre-registered.** Small ranges; optuna stays inside them. No free-roaming period optimization.
- **Regime gate optional.** Most Tier I are trend/breakout flavoured → gate with A4/B2/B5 to suppress chop.
- **Weak-proxy by default** on every result page.

## 3. Strategy specs

### I1 — EMA ribbon + RSI regime filter (both; 15m + daily)
- **Hypothesis:** stacked EMAs (8/21/55) define trend; RSI in a trend band (>50 up) filters false flips. **Entry:** EMAs stacked in order + price above ribbon + RSI in band. **Exit:** ribbon cross-down OR RSI exits band OR stop/time. Bounds: EMA periods within {5..13},{18..30},{45..70}; RSI band edges {45..55}. **Fails when:** choppy regime whipsaws the ribbon.

### I2 — CPR width regime + pivot reaction (both; 15m + daily)
- **Hypothesis:** Central Pivot Range (pivot, BC, TC from prior session) — **narrow** CPR → trend day, **wide** CPR → range day; price reacts at pivot/S-R. **Entry:** narrow-CPR days → breakout from CPR with confirmation; wide-CPR days → fade R1/S1. **Exit:** opposite pivot level OR stop OR session close. Bounds: CPR-width percentile threshold {20..40}. **Fails when:** gaps invalidate levels; **test whether CPR width has any real predictive content — it may not.**

### I3 — Volume-spike + price/RSI confirmation (both; 15m + daily) — **stock cash / index futures**
- **Hypothesis:** an abnormal volume spike with directional price confirms participation. **Entry:** volume ≥ k× avg + price breaks prior bar range + RSI confirming. **Exit:** target R-multiple OR volume fades OR stop. Bounds: k∈{1.8..3.0}, RSI confirm {50..60}. **Fails when:** spike is news-driven and reverses; illiquid F&O stocks bleed slippage. Volume = participation is true on stocks' cash, NOT on index spot (use futures there).

### I4 — OI buildup + price + MA combo (both; daily) — **index/stock futures OI**
- **Hypothesis:** rising OI + rising price above MA (long buildup) = fresh longs; rising OI + falling price below MA (short buildup) = fresh shorts. **Entry:** long-buildup quadrant + price above MA. **Exit:** quadrant flips OR price loses MA OR stop/time. Bounds: MA period {20..50}, OI-change threshold pre-registered. **Assumed (state honestly):** OI grid maps to direction — a heuristic, not identity; cannot reveal aggressor/hedge-leg. **Fails when:** OI change is arb/hedge near expiry.

### I5 — VWAP + RSI cross momentum (intraday; 5m/15m) — **stock cash / index futures**
- **Hypothesis:** price reclaiming/holding session VWAP with RSI crossing 50 marks intraday momentum. **Entry:** reclaim VWAP + RSI crosses 50 up (long) / down (short). **Exit:** VWAP loss OR RSI re-cross OR session close. Bounds: RSI length {9..14}. **Fails when:** low-volume drift; VWAP magnet in range.

### I6 — Triple-confirmation (Supertrend + EMA + RSI all agree) (both; daily) — **FALSIFICATION TARGET**
- **Hypothesis:** requiring three indicators to agree cuts false signals. **Honest note:** stacking confirmations usually **reduces trade count and lags entries** → expect mediocre risk-adjusted results after costs. **Included precisely to test the "more confirmation = better" folklore.** **Entry:** Supertrend long + price above EMA + RSI>50 (all true). **Exit:** any one flips OR stop/time. Bounds: each indicator's standard small range.

### I7 — Pivot / S-R bounce + volume confirmation (intraday; 5m/15m) — **stock cash / index futures**
- **Hypothesis:** price bounces from classical pivot S1/R1 with supporting volume. **Honest note:** high hindsight-fit risk (levels chosen post-hoc look perfect). Levels are mechanical (prior-session pivots), volume confirms. **Entry:** touch S1/R1 + reversal bar + volume ≥ avg. **Exit:** next pivot level OR stop OR session close. Bounds: volume-confirm multiple {1.2..2.0}. **Fails when:** levels break in trends; bounces are coincidental.

### I8 — Bollinger squeeze → breakout (NEW) (both; 15m + daily)
- **Hypothesis:** a **volatility squeeze** (Bollinger Bands inside Keltner Channels, i.e. band-width at a multi-period low) precedes an expansion/breakout — the squeeze is a *real* volatility-regime signal, unlike most band trades. **Entry:** on squeeze release, enter in the breakout direction (price closes outside the upper/lower band with momentum confirmation). **Exit:** band mid-line revert OR ATR stop OR time. Bounds: BB period {18..22}, BB mult {1.8..2.2}, Keltner mult {1.2..1.8}, squeeze-lookback {6..12}. **Falsifies the related folklore** that mean-reversion at the bands ("buy lower band / sell upper band") works — include a **mean-reversion variant** alongside the breakout variant and let the data say which (if either) survives. **Fails when:** false breakouts in chop; whipsaw on band touches.

### I9 — RSI(2) Connors mean-reversion (NEW) (both; daily) — uses execution realism
- **Hypothesis:** a *specific, published* short-term mean-reversion rule (Connors & Alvarez): in an uptrend (price > 200-SMA), buy when RSI(2) < a low threshold (e.g. 5–10), exit when price closes above a short MA or RSI(2) rises above a mid level. More precise and more falsifiable than vague "RSI oversold." **Entry:** price > long-SMA AND RSI(2) < low_thr. **Exit:** close > short-SMA, OR RSI(2) > mid_thr, OR stop, OR time. Bounds: RSI length {2..3}, low_thr {5..15}, long-SMA {150..200}, short-SMA {5..10}. **Mandatory:** the engine's mean-reversion spread-crossing penalty and next-bar fills — RSI(2) systems look great on close-to-close backtests and degrade sharply with realistic fills/costs. **Fails when:** trend breaks during the dip (catching a falling knife); cost drag from frequent small trades.

## 4. Per-strategy result page (same template as Phases 3–4)

Name · type · required data · instruments · timeframes · logic · explicit entry/exit/risk · regime assumptions · why it may work · why it may fail · traps/false positives · **weak-proxy** label (Tier I default) · feasibility · full metrics + per-regime buckets + ₹1-lakh table + sensitivity + per-year (Phase 6) · **beats-null flag.**

## 5. DEFINITION OF DONE

- [ ] preregistration.json extended with I1–I9 (bounded ranges, instruments, data_kind, rationale).
- [ ] Volume strategies use cash_equity on stocks and futures on index; OI strategies use futures OI; engine guard passes.
- [ ] I8 implements BOTH a breakout and a mean-reversion variant; I9 uses the mandatory mean-reversion execution model.
- [ ] Each runs once on Train+Val (one index + a few stocks) with a finite report incl. per-regime buckets.
- [ ] Each carries a "beats N1/N2/N3/N4 net of costs?" flag.
- [ ] No indicator period escapes its pre-registered bounds.

## 6. Hand to Phase 6

Tier I implemented and runnable on indices + stocks. Phase 6 runs the full sweep, builds the all-timeframe ₹1-lakh dashboard with regime buckets, assembles the Tier M ensemble, runs SPA/FDR, and reports honestly which Tier I combos (if any) survive.
