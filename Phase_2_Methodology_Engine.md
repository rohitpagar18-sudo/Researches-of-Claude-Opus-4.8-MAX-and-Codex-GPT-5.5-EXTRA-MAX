# PHASE 2 — Methodology Engine & Strategy Base Class (v3)

> Feed after Phase 1 passes. This is the anti-overfitting scaffolding. **No strategy runs until this exists.**

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phase 1 (data layer incl. F&O stock universe, cross-sectional panel, intraday-stitched futures, costs.py)
is complete and passed. Build the methodology engine + StrategyBase per this Phase 2 doc, BEFORE any
strategy. Build:
1. StrategyBase abstract class with a uniform interface (asset-class-agnostic 'market' field).
2. Chronological Train/Val/Holdout split (~50/25/25) PER INSTRUMENT. Holdout LOCKED with a hard guard
   (reading it during tuning must raise).
3. Pre-registration writer -> preregistration.json BEFORE backtests.
4. Walk-forward engine (rolling re-fit; OOS is the headline) with purge+embargo support.
5. optuna tuner constrained to pre-registered bounds, on Train+Val only.
6. Metrics module: full set + Deflated Sharpe (trial count INCLUDES the instrument dimension) + a
   1-LAKH NORMALIZED block (start equity 100000) + PER-REGIME breakdown buckets.
7. EXECUTION-REALISM module: next-bar-open fills by default, spread-crossing penalty for mean-reversion,
   limit-fill probability for breakout entries, liquidity- and size-scaled slippage. Wired into every backtest.
8. MULTIPLE-TESTING module: (a) Deflated Sharpe, (b) White's Reality Check + Hansen's SPA via stationary
   block bootstrap, (c) Benjamini-Hochberg + Benjamini-Yekutieli FDR for the cross-sectional panel,
   (d) correlation/sector clustering -> effective number of independent bets.
9. An INSTRUMENT x STRATEGY x TIMEFRAME runner over NIFTY, BANKNIFTY, and the point-in-time F&O universe.
10. quick_screen() TRIAGE: fast lightweight backtest across timeframes, full costs, default/coarse params,
    NO optuna, NO holdout, stamped "unvalidated triage"; cannot set rankings.
11. Robustness module: parameter sensitivity, per-year + per-regime breakdown, <30-trade flag, cost
    sensitivity (1x vs 2x slippage), and a deflated-vs-raw delta.
12. Reproducibility: every backtest logs a config hash + RNG seed; results must reproduce bit-for-bit.
13. Dynamic-factor discipline (Section 8): allowed adaptivity vs forbidden adaptive knobs.

Wire so a dummy strategy on one index + one stock produces a full cost-netted, walk-forward, deflated,
1-lakh-normalized, per-regime report across its timeframes. Verify Definition of Done. No real strategies yet.
```

---

## 1. Objective

Build the uniform interface + rigor layer so every strategy plugs in identically and is judged honestly across **instruments, timeframes, and regimes**, with ₹1-lakh interpretability, realistic execution, an institutional-grade multiple-testing defense, and a fast triage path that cannot become a fishing tool.

## 2. `StrategyBase` interface

```python
class StrategyBase(ABC):
    tier: str            # NULL|A|B|C|D|E|I|M
    novelty: str         # Existing|Reframe|Novel
    direction: str       # intraday|positional|both|gate
    timeframes: list     # subset of {5m,15m,60m,day,week}
    instruments: str     # "index" | "stocks" | "both"
    market: str = "NSE"  # asset-class-agnostic seam (future: "CRYPTO","FX")
    cross_sectional: bool = False   # True for A7 / pair strategies that need the panel
    params: dict
    param_bounds: dict   # pre-registered bounded ranges
    data_kind: str       # "spot"|"futures"|"cash_equity"  (index volume->futures; stock volume->cash_equity)
    forward_test_only: bool = False

    @abstractmethod
    def generate_signals(self, data) -> "entries/exits": ...
    # Universal rules the runner ENFORCES (not optional):
    #  vol-targeted sizing; per-trade risk cap (default 1% equity, pre-registered NOT optimized);
    #  max concurrent exposure + max daily loss + optional max-trades/day hard caps;
    #  all exits (stop/target/time) defined before entry; execution-realism applied to every fill.
```

## 3. Splitting (`methodology/split.py`)

Chronological 50/25/25 **per instrument** (each stock and each index has its own locked Holdout). Hard guard: tuning code physically cannot read Holdout; the final-ranking path uses an explicit `final_ranking=True` flag, used once in Phase 6; accidental access raises. For cross-sectional strategies, the split is **by date** across the panel (same train/val/holdout date boundaries for all symbols) to avoid leakage through the ranking step.

## 4. Pre-registration

Before any backtest, write `{id, hypothesis, params, param_bounds, direction, timeframes, instruments, data_kind, cross_sectional, structural_rationale}` per strategy to `preregistration.json`. optuna only proposes within bounds, on Train+Val. Phase 6 reads the **total trial count** (instrument × strategy × timeframe × variant) to feed Deflated Sharpe and the reality-check penalty.

## 5. Walk-forward + metrics

Walk-forward: fit on window N (Train+Val), trade N+1, roll; report OOS as headline + IS-vs-OOS gap (overfit indicator). For ML/feature strategies (B4), use **purged + embargoed** splits (no feature leakage across the boundary).

**Full metric set per instrument × strategy × timeframe:** yearly P&L · max DD % + duration · win rate % · profit factor · best/worst year · CAGR · ann. vol · Sharpe · **Deflated Sharpe (rank on this)** · Sortino · Calmar · avg R-multiple + expectancy · trade count (+ <30 flag) · turnover + cost drag · OOS-vs-IS gap · **deflated-vs-raw Sharpe delta**.

**Per-regime breakdown (NEW):** every result is also bucketed into **bull / bear / chop** (trend regime) and **low-vol / high-vol** (VIX or realized-vol regime), with metrics per bucket, so the dashboard can show *where in the market cycle* a strategy earns and where it bleeds. This directly delivers "stress-test across trend / sideways / volatile."

**₹1-LAKH NORMALIZED block (interpretability, NOT for ranking):** start_equity ₹1,00,000, compounded; ending_equity, total_profit ₹, total_return_%, CAGR_%, max_dd_₹, max_dd_%, per-year ₹ P&L; **max_leverage_used** and **return_per_unit_risk** so a leveraged 300% can't masquerade as a clean 300%. For F&O, model realistic margin. **Ranking stays on Deflated Sharpe / Calmar; the ₹ block sits beside it, never as the rank key.**

## 6. Execution realism (NEW — the realism that kills fake intraday edges)

Wired into every backtest, configurable, logged:
- **Fills at next-bar open by default** — no filling at the signal bar's close (that is look-ahead). Strategies may declare a different, justified fill model in pre-registration.
- **Spread-crossing penalty for mean-reversion** — fading a move means crossing the spread at the worst moment; charge it. This is the single biggest reason retail intraday mean-reversion backtests overstate edge.
- **Limit-fill probability for breakouts** — a breakout entry at the level is not guaranteed; model partial/missed fills (e.g. require trade-through or a fill-probability haircut) rather than assuming a perfect fill at the breakout price.
- **Liquidity- & size-scaled slippage** — from `costs.py`; mid/small F&O stocks get wider slippage; large orders get impact.
- **Cost-sensitivity:** every result reported at 1× and 2× slippage; a strategy that only survives at 1× is flagged cost-fragile.

## 7. Instrument × strategy × timeframe runner (`engine/runner.py`)

- Iterate every **applicable** instrument (per `instruments`) × allowed timeframe (per the matrix). The point-in-time membership table decides which dates a stock is eligible. Cross-sectional strategies consume `/data/panel`.
- Pipeline: load correct data (`data_kind` enforced; index volume→futures, stock volume→cash_equity) → `generate_signals` → universal sizing/risk caps → **execution realism** → net costs (`costs.py`) → walk-forward → metrics (+₹1-lakh + per-regime) → robustness → write results (with config hash + seed).
- Tag sub-15-minute results with a low-sample warning and weight down in ranking.
- Any guardrail violation (look-ahead, Holdout touch, volume-on-spot, F&O stock used outside its eligibility window) → **INVALID**, excluded from ranking.

### Timeframe matrix (do NOT test all on all)
| Class | Test | Skip |
|-------|------|------|
| Positional trend/flow (A1, A6, A7, D1–D4, E1–E2) | Daily, weekly | sub-hour |
| Mean-reversion/pairs (A2, A3, I9) | 15m, 60m, daily | 1m |
| Intraday structure (C1, C3, C4, C5, B4, I5, I7) | 5m, 15m | daily |
| Both-capable (I1, I2, I6, I8, M1) | 15m, daily (+weekly where positional) | 1m |
| Regime gates (A4, B1–B3, B5, E3) | Daily | intraday |
| Vol premium (A5a) | Daily | intraday |

## 8. Dynamic-factor discipline ("works in any market", made safe)

"Performs in any market" is delivered ONLY by these allowed mechanisms:
- **Regime-gating** — enabled/disabled by A4 (VIX), B1 (HMM), B2 (Hurst), B5 (ADX), E3 (dispersion). A trend strategy goes flat in chop.
- **Volatility-targeted sizing** — size scales inversely with realized vol.
- **Walk-forward re-fitting** — parameters re-estimate each window; this *is* adaptation.
- **Ensemble rotation (Tier M)** — rotates across strategies by recent regime-conditional performance; sits small/flat when nothing has edge.

**Forbidden:** free "adaptive" knobs tuned on full history; any parameter the strategy adjusts using info from the test/holdout period; adding parameters to chase robustness (each one raises overfit risk and the Deflated-Sharpe penalty). **No single-strategy magic that claims all-weather edge — that does not exist; the honest all-weather construct is M1.**

## 9. Quick screener (`engine/quick_screen.py`) — triage only

`quick_screen(strategy, instrument, timeframes) -> compact table`: fast lightweight backtest (default/coarse params, full costs + execution realism, simple train/test, **no optuna, no Holdout**) across timeframes to surface where a strategy looks promising before the full pipeline.
- Output per timeframe: ₹1-lakh net return, CAGR, max DD%, win rate, profit factor, trade count, Sharpe — stamped **"UNVALIDATED TRIAGE — not for ranking."**
- **Hard rules:** never touches Holdout; never sets rankings; hits must still pass the full Phase 6 pipeline (walk-forward + bounded optuna + Deflated Sharpe + SPA/FDR + single Holdout). If the screener becomes the selector, it is just fast overfitting.

## 10. Multiple-testing — done right (the v3 upgrade)

The naive "rank by Sharpe across tens of thousands of runs" is a false-discovery machine. v3 requires all of:
1. **Deflated Sharpe** with the complete trial count (instruments included). Rank on deflated, not raw.
2. **White's Reality Check + Hansen's SPA** — using a **stationary block bootstrap** of returns, test whether the *best* strategy's OOS performance exceeds what data-snooping over the whole search would produce by chance. Report the SPA p-value for the top candidates. This is the institutional standard for "is the winner real?"
3. **FDR control across the cross-sectional panel** — Benjamini–Hochberg, and **Benjamini–Yekutieli** (handles dependence) for the correlated stock universe. Report "N combos passed raw threshold; ≈M expected by chance; FDR-controlled survivors = K."
4. **Correlation / sector clustering** — cluster survivors by return correlation and sector; collapse near-duplicates (e.g. many PSU banks on the same signal) and report the **effective number of independent bets**. A "winner" that is 1-of-15 correlated clones is one discovery, not fifteen.

Each per-instrument winner additionally clears that instrument's own locked Holdout (Phase 6).

## 11. Reproducibility

Every backtest logs: config hash (of params, data version, cost params, code commit), RNG seed, library versions. Re-running the same config reproduces the same result. The dashboard links each cell to its config hash.

## 12. DEFINITION OF DONE

- [ ] StrategyBase defined; dummy strategy runs end-to-end on one index + one stock with config hash + seed logged and reproducible.
- [ ] Split is per-instrument 50/25/25 (panel split by date); Holdout read during tuning raises.
- [ ] Pre-registration writer works; optuna refuses out-of-bounds values.
- [ ] Walk-forward returns OOS + IS-vs-OOS gap; purge+embargo available.
- [ ] Metrics include Deflated Sharpe (instrument-inclusive trial count), the ₹1-lakh block with max-leverage + return-per-unit-risk, AND per-regime buckets (bull/bear/chop, low/high-vol).
- [ ] Execution-realism module applies next-bar fills, spread-crossing penalty (mean-reversion), limit-fill haircut (breakouts), liquidity/size slippage; demonstrated on the dummy.
- [ ] Multiple-testing module computes Deflated Sharpe, SPA/Reality-Check (block bootstrap), BH + BY FDR, and correlation/sector clustering -> effective independent bets, on a toy set.
- [ ] Runner iterates instrument × strategy × timeframe; enforces data_kind and F&O-eligibility windows; marks violations INVALID.
- [ ] quick_screen() outputs the triage table stamped unvalidated; proven it cannot touch Holdout.
- [ ] Robustness returns sensitivity grid, per-year/per-regime tables, <30-trade + cost-fragility flags, deflated-vs-raw delta.
- [ ] Dynamic-factor discipline documented in code comments; no forbidden adaptive knobs present.

## 13. Hand to Phase 3

A working engine where implementing a strategy = subclass StrategyBase, declare bounds + instruments, done. Phases 3–5 write only strategy logic; all rigor, ₹-normalization, execution realism, multiple-testing, triage, and instrument iteration already live here.
