# PHASE 6 — Full Run, All-Timeframe Dashboard, Tier M Ensemble & Guarded Meta-Discovery (v3)

> Feed after Phase 5 passes. The only phase permitted to touch the locked Holdout — once per instrument.

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phases 1-5 are complete and passed. Execute the full research protocol per this Phase 6 doc:
1. Run the complete walk-forward + cost + execution-realism + robustness sweep across INSTRUMENT x
   STRATEGY x TIMEFRAME for every non-forward-test-only strategy (NIFTY, BANKNIFTY, point-in-time F&O
   universe), on Train+Val, tuning ONLY within pre-registered bounds. Log config hash + seed per cell.
2. Compute Deflated Sharpe with the TOTAL trial count (instruments included). Run White's Reality Check +
   Hansen's SPA (stationary block bootstrap) on the top candidates. Run BH + BY FDR across the
   cross-sectional stock panel. Cluster survivors by correlation/sector -> effective independent bets.
3. Touch each instrument's locked Holdout EXACTLY ONCE for final OOS numbers. Never tune on it.
4. Build the Tier M regime-aware ensemble from validated sub-strategies (walk-forwarded; allocation rule
   pre-registered and penalised).
5. Build the ranked HTML dashboard: every strategy shows ALL its timeframes side by side, a 1-LAKH P&L
   table per strategy x instrument x timeframe, PER-REGIME performance buckets, filters by tier/direction/
   timeframe/instrument-type/regime, "good in both intraday AND positional" flag, "beats buy-and-hold"
   flag, INVALID markers, cost-fragility + <30-trade flags, a cross-sectional FDR/SPA panel, and a separate
   forward-test-only section. Rank on Deflated Sharpe (Holdout OOS); show 1-lakh return + max leverage +
   max DD + Calmar beside it.
6. Write the data-quality/limitations log and the research summary (honest caveats; NO live recommendation).
7. THEN run the guarded meta-discovery loop (Section 7): pre-register, count in the trial penalty,
   FORWARD-TEST-ONLY until validated on unseen data, never mine Holdout.
Any guardrail violation -> INVALID, not ranked. Produce all deliverables. Verify Definition of Done.
```

---

## 1. Objective

Run the full protocol honestly across instruments, timeframes, and regimes; rank on risk-adjusted OOS with an institutional multiple-testing defense; surface where each edge lives (intraday vs positional, index vs stock, which regime); express the disciplined "all-weather" idea as Tier M; and run a fenced discovery loop.

## 2. The full sweep

- For each instrument × strategy × allowed timeframe: bounded optuna tuning → walk-forward → execution realism → cost-netting → robustness, on **Train+Val only**. Config hash + seed logged per cell (reproducible).
- Sub-15-minute results carry the low-sample warning and are weighted down. Data-thin cells (<30 trades) are flagged, not dropped.
- Forward-test-only items (A5b, intraday-chain variants, pre-open logic) run only on accumulated recorder data, in a **separate labeled section** — never ranked against historically-backtested strategies.

## 3. Multiple-testing — done right (the v3 panel)

- **Deflated Sharpe** with every instrument × strategy × timeframe × variant in the trial count (from preregistration.json). Rank on **deflated OOS**.
- **White's Reality Check + Hansen's SPA** (stationary block bootstrap) on the top candidates: report the SPA p-value — the probability the best performance is a data-snooping artifact. A low Deflated Sharpe rank with a poor SPA p-value is a red flag, surfaced not hidden.
- **FDR panel:** BH and BY across the cross-sectional stock panel — "N combos passed raw threshold; ≈M expected by chance; FDR-controlled survivors = K."
- **Clustering:** collapse correlated/sector-duplicate survivors; report the **effective number of independent bets** so a "winner" that is 1-of-15 clones counts once.
- Apply the **"beats N1 buy-and-hold net of costs"** screen everywhere; flag failures (most Tier I and C3 are expected to fail — report it).

## 4. Single Holdout evaluation (per instrument)

After tuning/selection is final on Train+Val, evaluate chosen configs **once** on each instrument's locked Holdout via `final_ranking=True`. Holdout numbers are the dashboard headline. If Holdout collapses vs Train+Val walk-forward → overfit; report it, do not re-tune. The access guard logs a single authorized touch per instrument.

## 5. Tier M — regime-aware ensemble (the disciplined "dynamic / all-weather" construct)

- **M1:** a portfolio that allocates across **validated** sub-strategies based on recent regime-conditional performance (regimes from A4/B1/B2/B5/E3), and **sits small or flat when no sub-strategy has edge.**
- **Discipline:** the allocation rule is itself a parameter set → pre-register it, walk-forward it, count it in the trial penalty. Build M1 **only** from sub-strategies that already passed their own Holdout. Validate M1 on a reserved future slice and/or forward (Phase 7).
- **Honest framing in the summary:** no single strategy is all-weather; M1 approximates it by rotating and standing aside, not by magic adaptation. Show M1's **regime-by-regime contribution** so the rotation is transparent. This is the honest answer to "strategies should adapt to different market conditions."

## 6. The dashboard

Single HTML + ranked CSV. Requirements:
- **Per strategy, all timeframes side by side** (5m/15m/60m/day/week as applicable) → see if it earns intraday, positional, or both. A **"good in both"** flag fires when risk-adjusted metrics are acceptable in ≥1 intraday AND ≥1 positional timeframe.
- **Per-regime performance** (bull/bear/chop, low/high-vol) for each strategy → see where in the cycle it earns vs bleeds (the "stress-test across trend/sideways/volatile" view).
- **Instrument dimension:** filter index vs stock; drill into per-stock results; an aggregate view across the universe.
- **₹1-lakh P&L table** per strategy × instrument × timeframe: ending equity ₹, total profit ₹, total return %, CAGR %, max DD ₹ and %, **per-year ₹ P&L**, max leverage used, return per unit risk.
- **Rank on Deflated Sharpe (Holdout OOS)**; show ₹-return, Calmar, max DD, and max-leverage **beside** the rank so return is never read in isolation.
- Filters: tier · direction (intraday/positional/both) · timeframe · instrument type · regime · "beats null" · "good in both".
- **Flags surfaced, not hidden:** IS-vs-OOS gap, cost sensitivity (1× vs 2×), <30-trade, "does not beat buy-and-hold", INVALID (with reason: look-ahead / Holdout-touch / volume-on-spot / out-of-eligibility-window), and the **SPA p-value** for top candidates.
- Per-strategy equity-curve + drawdown charts; regime-shaded.
- **Cross-sectional FDR/SPA panel** visible on the stock-results view (raw passes vs expected-by-chance vs FDR survivors vs effective independent bets).
- **Separate forward-test-only section**, clearly labeled.

## 7. Guarded meta-discovery loop ("Claude as scientist")

Permitted; the naive version is overfitting. Rules:
1. **Mechanism-based hypotheses**, formed from Train+Val results/data, each with a stated structural rationale — not a backtest peak.
2. **Pre-register** each before testing; recompute the Deflated-Sharpe penalty and the reality-check set for the WHOLE study to include them.
3. **Never touch Holdout** to discover or validate — Holdout is already spent (Section 4) and off-limits to anything derived after seeing data.
4. **Forward-test-only:** validate on data generated *after* discovery (reserved future slice and/or chain recorder). Appears in the dashboard only in the forward-test section with that label — never ranked among historically-validated strategies.
5. **Report the search:** how many hypotheses generated vs survived, so selection pressure is visible.

This is where the "let the AI analyze the data like a researcher and propose new strategies" goal lives — fenced so it cannot become invisible overfitting.

## 8. Deliverables (all)

1. **Codebase** — modular, documented (Phase 0 repo layout); config-hash + seed reproducibility.
2. **preregistration.json** — all hypotheses + bounds incl. meta-discovered, timestamps, final trial count.
3. **Per-strategy result pages** — entry/exit/risk + full metrics + per-regime buckets + ₹1-lakh table + sensitivity + per-year + fact/assumption/unknown (D/E) + feasibility + beats-null flag.
4. **Ranked dashboard** (HTML) + ranked CSV — per Section 6.
5. **Multiple-testing report** — Deflated Sharpe, SPA/Reality-Check p-values, BH/BY FDR panel, clustering / effective independent bets.
6. **Data-quality & limitations log** — fetched/cached/blocked/suspect; forward-test-only items; statistically-thin items; the stock-universe survivorship/false-discovery notes; provisional-revision handling; corp-action handling; intraday-futures-stitch coverage.
7. **Research summary** — top candidates by Deflated Holdout OOS, each WITH honest caveats (regime dependence, sample size, cost sensitivity, assumption fragility, SPA p-value, and for per-stock winners where they sit vs the FDR floor). Explicit: **NO live-trading recommendation; selection is the human's decision; forward paper-testing (Phase 7) is the required next bridge.**

## 9. DEFINITION OF DONE

- [ ] Full sweep run across instruments × strategies × timeframes on Train+Val; tuning stayed in bounds; cells reproducible (config hash + seed).
- [ ] Deflated Sharpe with the complete (instrument-inclusive) trial count; raw vs deflated shown; SPA/Reality-Check p-values for top candidates; BH/BY FDR panel; clustering → effective independent bets.
- [ ] Each instrument's Holdout evaluated exactly once; access guard logged a single authorized touch per instrument.
- [ ] Tier M built only from Holdout-passed sub-strategies; allocation rule pre-registered + penalised; regime contribution shown.
- [ ] Dashboard shows all timeframes per strategy, per-regime buckets, the ₹1-lakh tables, instrument filter, "good in both" + "beats null" flags, INVALID markers, SPA panel, forward-test section, and charts. Ranked on Deflated Sharpe with return/leverage/DD/Calmar beside it.
- [ ] Limitations log + research summary written with caveats and the no-recommendation statement.
- [ ] Meta-discovery (if run) followed all five rules; discovered items forward-test-only and counted.
- [ ] No guardrail violation is ranked; every violation marked INVALID with a reason.

## 10. Hand to Phase 7

You review the dashboard and select a small set of top performers (on risk-adjusted metrics, with where-it-works clear: which instrument, which timeframe, intraday/positional/both, which regime). Phase 7 forward-validates them before any alerting is considered.
