# PHASE 4 — Practitioner (C), FII/DII Crowding (D) & Novel Hypotheses (E) (v3)

> Feed after Phase 3 passes. These consume the Phase 3 gates and the Phase 1 NSE/futures data.

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phase 3 (benchmarks, Tier A incl. A6/A7, Tier B gates incl. B5) is complete and passed. Implement
Tier C (practitioner, including the v3 addition C5 gap strategy), Tier D (FII/DII reframed as CROWDING,
not flow-following), and Tier E (novel) per this Phase 4 doc. For each: pre-register hypothesis +
structural rationale + bounded ranges, subclass StrategyBase with explicit entry/exit/risk, set data_kind
(futures for all volume/VWAP/POC logic; cash_equity for stock volume; stocks for gap), and wire any
consumed gates from Phase 3. Respect every "feature-only"/"gate-only"/"forward-test-only"/"partially-
testable"/"falsification-target" label exactly — do NOT promote a confirming feature to a standalone
ranked strategy. Run each once on Train+Validation to confirm a finite report. Full sweep is Phase 6.
Verify the Definition of Done.
```

---

## 1. Objective

Add practitioner techniques (tested skeptically), the FII/DII strategies reframed as crowding/positioning extremes, and the novel research hypotheses — each with honest feasibility labels.

## 2. The interpretation rule for Tier D (read before coding any D strategy)

FII/DII public data is **aggregate, category-level, end-of-day levels and their day-over-day flows** — that is the entire observable surface. Moving from that to "institutions are positioning for direction X" requires assumptions that are often false: category coherence (FII is not one agent), directionality (much index-futures activity is **cash-futures basis arbitrage** — a hedge, not a view), the OI-grid heuristic (not an identity), and delivery=conviction (conflates rebalancing/passive flows). **Therefore:** build **crowding / positioning-extreme** signals (COT-style), used **contrarian at percentile extremes**, with flow agreement only as a **confidence gate**. Tier E pushes further to the **futures basis**, microstructurally cleaner than raw OI. Every D result page states its fact / assumption / unknown split.

---

## 3. Tier C — practitioner techniques (test skeptically; expect rejections)

### C1 — Volume Profile / POC reversion (both; 5m/15m/60m) — **FUTURES / stock-cash**
- **Hypothesis:** price reverts to the prior-session high-volume node (POC). **Data:** futures (index) or cash (stocks). **Entry:** price extends beyond a band from POC, fade toward POC. **Exit:** reach POC, OR stop, OR time-stop. **Status:** modest real edge if cost-controlled. **Trap:** POC computed with any future bars = look-ahead; use only the completed prior-session profile.

### C2 — Anchored VWAP reaction (both; 15m/60m/daily) — **FUTURES / stock-cash**
- **Hypothesis:** price reacts at an institutional execution benchmark (VWAP anchored to a session/event). **Entry:** defined reaction (rejection/reclaim) at the anchored VWAP. **Exit:** target/stop/time. **Status:** logic sound, NSE edge unproven. **Trap:** anchor chosen with hindsight; pre-register the anchor rule. (Optional: VWAP ± standard-deviation bands as a variant.)

### C3 — SMC: order block / break-of-structure / liquidity sweep (both; 5m/15m) — **FALSIFICATION TARGET**
- **Hypothesis:** institutions defend zones / sweep stops then reverse. **Status:** very popular on YouTube, little rigorous evidence, **HIGH hindsight-fit risk.** Treat this as a **falsification target**: mechanize zone/BoS/sweep definitions with ZERO discretion and zero forward-looking bars, then report honestly whether it beats nulls. **Expect rejection.** Showing it fails with data is the deliverable — do not widen the search to rescue it. Lowest priority / highest mechanization effort; if time-bound, run a single mechanical definition rather than many.

### C4 — Opening Range Breakout (intraday; 5m/15m) — **FUTURES / stock-cash**
- **Hypothesis:** the first X minutes frame the day; a clean break with confirmation continues. **Entry:** break of OR high/low after the OR window (X∈{15,30,45}min, pre-registered) with volume/ATR confirmation — **modeled with the engine's breakout fill-probability haircut** (you do not always get filled at the level). **Exit:** opposite-side stop, R-multiple target, hard session-close flat. **Status:** clean and mechanical but regime-dependent (trends yes, chop bleeds) → gate with A4/B2/B5. **Trap:** survivorship of "good ORB days"; test on all days.

### C5 — Gap-fill / gap-and-go (NEW) (both; daily + 5m/15m open) — **STOCKS (cash)**
- **Hypothesis:** overnight gaps in single stocks resolve in two characteristic ways — small/unsupported gaps **fade (fill)** toward the prior close; large/volume-backed gaps **continue (gap-and-go)**. Stocks gap far more than the index (earnings, news, block deals), so this is a **stock-universe** strategy with no index analogue. **Signal:** classify the opening gap by size (vs ATR) and early volume; **fade** small low-volume gaps toward prior close, **follow** large high-volume gaps on a first-pullback or opening-range break. **Exit:** prior close (fade) / R-multiple (go), stop, hard session-close flat. Bounds: gap-size threshold (ATR multiples), early-volume multiple, fade-vs-go cutoff — pre-registered. **Trap:** event-driven gaps (results) are bimodal and risky — flag earnings-day gaps separately (use the event calendar from E4); slippage on illiquid names eats the edge.

---

## 4. Tier D — FII/DII reframed as crowding

### D1 — FII futures positioning extreme (positional, daily) — most defensible D — **INDEX ONLY**
- **Feature:** FII net index-futures position as a rolling percentile (lookback pre-registered, e.g. 252d), point-in-time. **Entry (contrarian):** percentile > upper extreme (e.g. 90th) → fade-short bias; < lower extreme (e.g. 10th) → fade-long bias. **Direction and thresholds are pre-registered and TESTED, never assumed.** **Confirmation:** optionally D2. **Exit:** percentile mean-reverts past midline, OR time-stop, OR stop. Interpretation locked to crowding, not flow-following.

### D2 — FII cash + futures convergence (GATE, daily) — **INDEX ONLY**
- Agreement of two independent streams (cash net + futures positioning) as a **confidence gate** on D1 / positional longs. Not a primary signal.

### D3 — OI buildup / unwinding (CONFIRMING FEATURE only; daily/15m) — **index + stocks**
- Long-buildup / short-covering / short-buildup / long-unwinding grid as a **confirming feature** inside other strategies. **Never standalone** — the grid is a heuristic, not an identity (cannot reveal aggressor/initiator/hedge-leg). Stock OI from bhavcopy/stock futures; index OI from index futures.

### D4 — Delivery-% spike + FII cash (FILTER, daily) — **index + stocks**
- High delivery% + supportive FII cash as a **filter** on other longs. Weak alone; delivery% conflates rebalancing/passive/block settlement with conviction. Stock delivery% from bhavcopy.

---

## 5. Tier E — novel research hypotheses (disciplined)

### E1 — Futures basis / roll-yield signal (positional, daily) — **index + stocks(futures)**
- **Feature:** annualized basis = f(futures, spot, days-to-expiry) using the **unadjusted** per-contract futures + spot + the point-in-time expiry calendar; build a roll-yield series. **Hypothesis:** basis deviation reflects directional futures demand **net of arbitrage**, cleaner than raw OI. **Entry:** basis deviates from rolling norm beyond a pre-registered band. **Exit:** basis normalizes, OR time-stop near expiry, OR stop. Handle near-expiry explicitly (basis → 0 mechanically).

### E2 — Overnight vs intraday return split × FII cash (positional+intraday; daily decomposition) — **INDEX**
- **Hypothesis:** close-to-open and open-to-close returns have different drivers; FII cash (a session-linked flow) ties to the split. Decompose daily returns; condition positioning on the overnight/intraday asymmetry and FII cash sign. **Trap:** provisional FII cash is revised — use as-of values.

### E3 — Index dispersion regime (GATE, daily) — **PARTIALLY TESTABLE**
- **Hypothesis:** low constituent dispersion → index trends; high → chops (and high dispersion favours cross-sectional momentum A7). Gate trend vs reversion. **Survivorship caveat:** today's NIFTY-50 names on history is survivorship bias → **route to a sector-index dispersion proxy** (cross-sectional dispersion of NSE sector indices), free and reasonably point-in-time. Label "Partially testable"; document the proxy's limits.

### E4 — Scheduled-event drift windows (both; daily + intraday around events) — **index + stocks**
- **Hypothesis:** positioning drift around RBI policy, Union Budget, **expiry** (Tuesday now, Thursday pre-Sep-2025; BANKNIFTY weekly gone post-Nov-2024), and **single-stock earnings** (for the gap interaction in C5). Build a **scheduled** event calendar; measure drift in fixed pre/post windows. **Trap:** few events = low sample → flag; event dates must be the *scheduled* (known-in-advance) dates, not outcomes.

---

## 6. Feasibility labels (on every result page)

Fully testable · Partially testable · Proxy-only · Forward-test-only. In this phase: C1/C2/C4/C5 fully testable (intraday history now multi-year via pagination, but recently-added stocks still thin → low-sample flag); C3 fully testable but **falsification target** (high rejection odds); D1/D2 index-only daily; D3/D4 daily incl. stocks; E1/E2/E4 fully testable on daily EOD; E3 partially testable (proxy). Anything needing intraday option chain = forward-test-only via the recorder.

## 7. DEFINITION OF DONE

- [ ] `preregistration.json` extended with C1–C5, D1–D4, E1–E4 (bounded ranges + rationale).
- [ ] All volume/VWAP/POC/ORB strategies have correct `data_kind` (futures on index, cash on stocks); engine guard passes.
- [ ] C5 gap strategy is stock-only, classifies gaps by size+volume, flags earnings-day gaps via the E4 calendar.
- [ ] D strategies implemented strictly as crowding/gate/feature/filter per labels (D3 not standalone; D1/D2 index-only).
- [ ] E1 uses unadjusted futures + expiry calendar for basis; near-expiry handled.
- [ ] E3 uses the sector-proxy, labeled partially testable.
- [ ] E4 uses the point-in-time expiry calendar + scheduled event dates.
- [ ] Each runs once on Train+Val through the engine with a finite report incl. per-regime buckets.

## 8. Hand to Phase 5

Practitioner, crowding, and novel tiers implemented and individually runnable. Phase 5 adds the Tier I indicator combinations on indices + stocks.
