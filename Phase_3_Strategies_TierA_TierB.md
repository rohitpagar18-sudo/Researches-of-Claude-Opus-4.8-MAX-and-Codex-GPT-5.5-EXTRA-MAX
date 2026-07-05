# PHASE 3 — Benchmarks (NULL), Established (Tier A) & Regime Gates (Tier B) (v3)

> Feed after Phase 2 passes. Implement benchmarks and regime gates first, because Tier A strategies may be gated by them.

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phase 2 (methodology engine + StrategyBase, execution realism, SPA/FDR) is complete and passed. Implement
the NULL benchmarks, Tier A established strategies (including the v3 additions A6 Donchian, A7 cross-sectional
momentum, and stock-pair stat-arb under A3), and Tier B regime gates (including B5 ADX) per this Phase 3 doc.
For EACH strategy:
 - first append its hypothesis + structural rationale + bounded parameter ranges to preregistration.json,
 - then subclass StrategyBase with generate_signals + explicit entry/exit/risk rules,
 - declare data_kind correctly (futures for any volume logic; cross_sectional=True for A7 and stock pairs),
 - register its allowed timeframes from the matrix.
Implement regime gates (A4, B1, B2, B3, B5) BEFORE the strategies that consume them.
Do NOT run the full sweep yet (Phase 6). DO run each strategy once on Train+Validation to confirm it produces
signals, trades, and a finite metric report (incl. per-regime buckets) through the engine. Verify Definition of Done.
```

---

## 1. Objective

Populate the roster's foundation: a null/benchmark tier every real strategy must beat, the established edges (Tier A — now including the cross-sectional and breakout families that use the stock universe), and the regime gates (Tier B) that modify them.

## 2. Universal rules (enforced by the engine; restate per strategy)

Vol-targeted sizing; per-trade risk cap (default 1% equity, pre-registered not optimized); max concurrent exposure + max daily loss + optional max-trades/day hard caps; all exits (stop/target/time) defined before entry; execution realism on every fill. No discretionary holds.

---

## 3. NULL tier — benchmarks (implement, but NEVER rank as competitors)

These exist so we know whether sophistication earns its keep. Any Tier A–I strategy that fails to beat **N1 net of costs** on the same period is flagged "does not beat null."

- **N1 Buy & hold** — long index (via continuous futures, rolled) and per-stock; daily. No params.
- **N2 EMA crossover (+ATR filter)** — fast/slow EMA cross; ATR filter to suppress chop. Bounds: fast∈{5..20}, slow∈{20..100}, atr_mult∈{0.5..2.0}. Daily + 15m. The "EMA crossover" idea lives here — as a baseline, not a candidate.
- **N3 Supertrend** — standard ATR-band trend follow. Bounds: period∈{7..14}, mult∈{1.5..3.5}. Daily + 15m.
- **N4 MACD (NEW)** — MACD line/signal crossover + histogram zero-cross. Bounds: fast∈{8..15}, slow∈{20..30}, signal∈{5..12}. Daily + 15m. MACD is mathematically a smoothed EMA-difference, so it belongs as a **baseline**, not a discovery — included because it is one of the most-quoted retail signals and must be measured against buy-and-hold.

> RSI divergence is intentionally NOT a standalone anywhere — hindsight-prone and hard to define without look-ahead. It may appear only as one input feature inside B4.

---

## 4. Tier A — established strategies

### A1 — Time-series (absolute) momentum (positional, daily/weekly)
- **Hypothesis:** sign of trailing K-period return predicts next-period sign (Moskowitz, Ooi & Pedersen 2012); robust across markets incl. India.
- **Signal:** sign of trailing K-period return, optionally vol-scaled. K∈{63,126,252} trading days (pre-registered). **Entry:** next open after flip. **Exit:** flip, OR vol-stop, OR time-stop. **Gate:** A4/B1/B5 may veto.
- **Why it may fail:** trend crashes / sharp reversals; crowded. **Trap:** look-ahead in the return window; survivorship in long history.

### A2 — Short-horizon reversal after extreme moves (both; 15m/60m/daily)
- **Hypothesis:** sharp 1–3 bar moves partially revert.
- **Signal:** entry when a normalized move exceeds a pre-registered z/ATR threshold; fade it. **Exit:** revert to mean, OR stop, OR time-stop.
- **Honest status:** edge has decayed; survives only net of realistic costs + a regime filter **and the spread-crossing penalty** (this is the strategy most likely to show fake intraday edge — the engine's mean-reversion execution model is mandatory here). **Trap:** fading genuine trend days → ruin; needs A4/B2 gate.

### A3 — Cointegration pairs (both; 15m/60m/daily) — **index pair + stock pairs (NEW breadth)**
- **Hypothesis:** a cointegrated spread is mean-reverting while cointegration holds.
- **Index pair:** NIFTY–BANKNIFTY. **Caveat:** the two are very highly correlated and BANKNIFTY is a large NIFTY component, so the "spread" is near a single factor and can snap around banking shocks. Size legs dollar-neutral with **point-in-time lot sizes** (NIFTY 65, BANKNIFTY 30 currently) and net cost per leg.
- **Stock pairs (NEW):** scan the cross-sectional panel for cointegrated **same-period, preferably same-sector** pairs (e.g. two private banks, two IT majors). This is a far richer, more robust stat-arb space than the single index pair and uses the universe's breadth.
- **Discipline (both):** rolling ADF on residual / Johansen must pass at pre-registered significance over the lookback; if it fails, the pair is OFF that period (point-in-time, no peeking). Hedge ratio Kalman-estimated, time-varying. **Entry:** spread z beyond ±Z_enter. **Exit:** z→0, OR ±Z_stop, OR cointegration breaks. **Multiple-testing:** the pair scan itself is a search — every tested pair counts in the trial penalty, and survivors go through SPA/FDR.

### A4 — VIX-conditioned regime switch (GATE, daily)
- Master filter, not standalone. Classify realized/implied-vol regime (India VIX levels/percentiles + realized vol) into low/normal/high; expose a state series enabling/disabling sub-strategies. **Bounds:** VIX percentile thresholds (pre-registered). **Trap:** regime boundaries fit in-sample; require OOS stability.

### A5a — Vol-risk-premium SIGNAL (positional, daily) — TESTABLE
- **Hypothesis:** implied (India VIX) persistently exceeds realized; the premium is informative. **Signal:** VRP = VIX_implied − realized_vol (20d close-to-close or Yang–Zhang); trade as directional/vol-regime positioning or as a gate input. **Exit:** VRP normalizes, OR stop, OR time-stop. **Why split:** needs no option premiums → backtestable.

### A5b — VRP options-selling implementation (positional, daily) — FORWARD-TEST-ONLY
- The premium-selling structure needs historical option premiums free sources cannot backfill. `forward_test_only=True`; accumulates via the Phase 1 chain recorder. **Do not rank against historically-backtested strategies.** Tail-risk heavy → hard risk caps + defined-risk structures only.

### A6 — Donchian / Turtle breakout (NEW) (positional, daily/weekly)
- **Hypothesis:** breakouts of the N-day high/low capture the start of trends — the classic Turtle/Donchian system, with real published pedigree (a genuine evidenced trend strategy missing from v2).
- **Signal:** enter long on close/break above the highest high of the last N bars; short below the lowest low (or long-only on stocks where shorting cash is constrained — declare in pre-registration). **Exit:** opposite M-day channel (M<N), OR ATR trailing stop, OR time-stop. Bounds: N∈{20,55}, M∈{10,20}, atr_stop∈{1.5..3.0}. **Gate:** B5 (ADX) / B2 (Hurst) to avoid chop. **Trap:** whipsaw in range markets; survivorship in long stock history.

### A7 — Cross-sectional momentum / relative strength (NEW) (positional, daily/weekly) — **STOCKS ONLY**
- **Hypothesis:** over 3–12 month formation, recent relative winners continue to outperform recent relative losers (Jegadeesh–Titman 1993; one of the most robust, widely-replicated equity anomalies). **This is the single strongest addition and only works on the stock universe — it cannot run on a lone index.**
- **Signal:** each rebalance (e.g. monthly), rank the eligible F&O universe by trailing K-month return (skip the most recent month to avoid short-term reversal); **long the top decile/quintile, short the bottom** (or long-only top decile if shorting is constrained — declare). Equal-weight or vol-weight legs. **Exit:** at rebalance when a name leaves the long/short set, OR vol-stop. Bounds: formation K∈{63,126,252}d, skip∈{0,21}d, top/bottom fraction∈{0.1,0.2}, rebalance∈{monthly,fortnightly}.
- **Discipline:** point-in-time F&O membership for the universe at each rebalance (no survivorship); panel split by date (no leakage through ranking); costs + execution realism on every leg; **turnover is high → cost drag is the make-or-break** (report it prominently). **Gate:** dispersion (E3) — cross-sectional momentum works better when dispersion is high. **Why it may fail:** momentum crashes (sharp reversals after bear-market bottoms); crowding; turnover cost.

---

## 5. Tier B — regime gates / method tools (gates, not standalone)

### B1 — HMM 3-state regime (gate, daily)
- Latent bull/bear/chop via hmmlearn (Hamilton 1989 lineage). **Discipline:** retrain on schedule; states stable out-of-sample (label-switching handled); fit only on Train+Val per walk-forward window — no peeking.

### B2 — Hurst exponent regime (gate, daily, rolling 60–120d)
- H>0.5 trending → enable trend strategies; H<0.5 mean-reverting → enable reversion. Cheap, sensible, weak alone. Extends to stocks via the panel.

### B3 — Kalman trend filter (gate/feature, daily/60m)
- Low-lag trend-state estimate via pykalman. Gate or feature.

### B5 — ADX / DMI trend-strength (NEW) (gate, daily)
- **Role:** classic trend-strength filter — ADX above a threshold (e.g. 20–25) flags a trending regime; below, a chop regime. Gates trend/breakout strategies (A1, A6, ORB, ribbon combos) so they go flat in range markets. Cheap, popular, and a sensible complement to Hurst. Bounds: ADX period∈{10..20}, threshold∈{18..28}. Weak alone; used as a gate/filter only.

### B4 — Gradient-boosted classifier (intraday, 5m/15m)
- Multi-feature next-bar/next-N-bar direction. Features may include RSI divergence, ATR, volume (futures), OI deltas, time-of-day, regime state. **HIGHEST overfit risk.** Mandatory strict walk-forward with **purged/embargoed** splits; if it cannot show stable OOS lift over N2/N3/N4 benchmarks, it is excluded. **Trap:** leakage via feature engineering across the split boundary — purge and embargo.

---

## 6. Per-strategy result page (produce for every strategy here and in Phases 4–5)

Name · type (intraday/positional/both/gate) · required data · instruments · test timeframes · logic summary · explicit entry/exit/risk rules · regime assumptions · why it may work · why it may fail · common traps/false positives · strong-evidence vs weak-proxy label · backtest feasibility · the full metric set + per-regime buckets + ₹1-lakh table + parameter-sensitivity + per-year table (filled in Phase 6).

## 7. DEFINITION OF DONE

- [ ] `preregistration.json` contains N1–N4, A1–A7 (incl. stock-pair config under A3), B1–B5 with bounded ranges + structural rationale.
- [ ] Each subclasses `StrategyBase` with explicit entry/exit/risk and correct `data_kind`; A7 and stock pairs set `cross_sectional=True`.
- [ ] Gates A4, B1, B2, B3, B5 implemented and expose state series consumed by ≥1 strategy.
- [ ] A5b flagged `forward_test_only`.
- [ ] A7 uses point-in-time F&O membership at each rebalance and the date-aligned panel; turnover/cost drag reported.
- [ ] A3 stock-pair scan counts every tested pair in the trial penalty.
- [ ] Each strategy runs once on Train+Val through the engine and yields a finite report incl. per-regime buckets (no run errors); volume-using logic confirmed on futures.
- [ ] B4 uses purged/embargoed walk-forward.

## 8. Hand to Phase 4

Foundation + gates live and individually runnable. Phase 4 adds practitioner, FII/DII-crowding, and novel strategies, several of which consume these gates.
