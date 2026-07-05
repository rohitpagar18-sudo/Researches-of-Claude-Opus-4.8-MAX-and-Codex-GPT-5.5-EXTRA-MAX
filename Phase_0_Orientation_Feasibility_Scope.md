# PHASE 0 — Orientation, Feasibility & Scope Decisions (READ THIS FIRST)

**Pack:** NSE Strategy Research System — Handoff Pack **v3** (Devil's-Advocate revision of v2).
**Target agent:** Claude Code / Codex (Opus 4.8 / GPT-5.x), Kite Connect **paid** API key in the same environment.
**Scope:** NSE — NIFTY 50, BANKNIFTY (index + continuous futures + recordable options) **AND the F&O-eligible STOCK universe (point-in-time, ~220 names — verify live).**
**Mode:** Research only. No live orders, no alerts, no auto-trading in this pack. The bridge to that is Phase 7.

> This phase has no build step. Read it, read all other phases, confirm the environment, and print a **feasibility report** (Definition of Done at the bottom) before touching Phase 1.

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
You are an autonomous quantitative research engineer. Before building anything, read THIS Phase 0
plus Phases 1-7 end to end. Then, WITHOUT writing strategy code yet:
1. Confirm the runtime environment: Python version, that a Kite Connect paid key + access token are
   present, available disk (the intraday cache will be tens of GB), and outbound network to NSE/Kite.
2. Verify-live the four load-bearing fact sets in this pack and print any drift vs the values quoted
   here: (a) Kite historical per-request caps + rate limits, (b) current index/stock lot sizes,
   (c) current STT/exchange/GST/stamp schedule, (d) current expiry-day rules and F&O membership count.
3. Print a FEASIBILITY REPORT: for each timeframe x instrument-class, the realistic history depth you
   can actually fetch, the expected number of usable trades, and which cells will be "data-thin"
   (flagged, not silently dropped). Print the estimated total historical-API request count and wall-clock
   fetch time at 3 req/s with caching.
4. Restate the scope decisions in Section 4 (NSE-only now; crypto/FX deferred) and the multiple-testing
   defense in Section 5 so it is explicit in your working memory.
Do not advance to Phase 1 until this report is printed and the Phase 0 Definition of Done passes.
Work autonomously; ask only if a fact cannot be verified or the environment is fundamentally missing.
```

---

## 1. The goal, stated plainly

Find a **small number of genuinely robust strategies** by testing many candidates across **indices + F&O stocks × multiple timeframes (intraday and positional)**, ranking on **risk-adjusted, out-of-sample** performance — never raw return. The dashboard shows every timeframe for every strategy on every instrument so we can see exactly *where* each edge lives (which instrument, which timeframe, intraday vs positional) and pick on drawdown-aware, cost-netted, overfitting-penalised metrics.

The eventual goal (Phase 7 and a later separate project) is paper-testing → alerts → auto-orders. **Backtest rank is a filter, not a guarantee.** Selection is the human's decision.

## 2. What this revision (v3) changes vs v2 — and why

The v2 pack was unusually rigorous and its hard facts were **verified correct** (lot sizes, STT, expiry rules all check out as of June 2026). v3 keeps all of that and fixes six things that materially affect whether the project succeeds:

1. **Intraday data availability was understated.** v2 repeatedly treats minute history as "~60 days." That 60-day figure is Kite's **per-request** cap, not total availability. With pagination you get **~3+ years of 1-minute and 5-minute data** and **20+ years of daily**. Intraday backtesting is far more feasible than v2 assumed — but the real intraday constraint moves elsewhere (continuous-futures stitching across expiries; see Phase 1).
2. **Multiple-testing defense upgraded.** Deflated Sharpe alone is not enough when ~220 *correlated* stocks × ~35 strategies × ~5 timeframes are tested. v3 adds **White's Reality Check / Hansen's SPA test** for the "is the best strategy real?" question, **Benjamini–Hochberg/Yekutieli FDR** for the cross-sectional panel, and **correlation/sector clustering** so a "winner" isn't just one of fifteen bank stocks catching the same beta (Phase 2 §).
3. **New strategies that use the breadth of the stock universe** — the biggest gap in v2, which only had time-series momentum and a single index pair. v3 adds **cross-sectional momentum / relative strength**, **stock-pair statistical arbitrage**, and **Donchian/Turtle breakout** (Phase 3), plus more retail indicators the user asked for (Phase 5).
4. **Execution realism** hardened: next-bar-open fills, spread-crossing penalty for mean-reversion, limit-fill probability for breakouts, and liquidity-scaled slippage for mid/small F&O stocks (Phase 2 §).
5. **A forward-validation bridge (Phase 7)** — the honest step between "ranked in a backtest" and "worth an alert."
6. **Cross-asset question answered** (Section 4): the *methodology* is asset-agnostic; the *data layer* and several strategy families are NSE-specific. Crypto/FX are deferred with a clean extension path, not bolted on now.

## 3. The danger this creates (read twice)

Testing ~220 stocks × ~35 strategies × ~5 timeframes ≈ **tens of thousands of backtests**. At 5% significance, ~5% look excellent by pure chance. "Best result on stock Y" is the **expected output of luck** unless it clears the multiple-testing penalty *and* a locked holdout *and* the reality-check panel. Worse, equities are cross-sectionally correlated: a true (or false) edge on one bank stock shows up on many, so naive "count the winners" both over- and under-counts. Every guardrail in this pack exists to separate signal from this noise. **A strategy that fails — including the popular YouTube indicator combos — is a valid, reportable result. Do not rescue it by widening the parameter search.**

## 4. Scope decisions (answers to the direct questions)

**"Can these strategies run on individual stocks, not just NIFTY/BANKNIFTY?"** — **Yes, and this pack is built for exactly that.** The F&O stock universe is a first-class instrument dimension (point-in-time membership). Several strategies (cross-sectional momentum, stock pairs, gap, delivery%/OI) **only make sense on the stock universe** and cannot run on an index. Some edges live on stocks and not the index, and vice-versa — the dashboard surfaces this.

**"Can we use them on Bitcoin or Forex?"** — **Not in this pack, and here is the honest reasoning:**
- **Data:** Kite/Zerodha does not provide crypto. NSE does not list spot crypto or spot FX. Crypto trades 24/7 on global venues (Binance/Coinbase via `ccxt`); spot FX is OTC/broker-dependent (OANDA, IC Markets, etc.). Different feeds, different cost models (maker/taker fees, perpetual **funding rates**, no STT/stamp), different microstructure (no overnight gap in crypto, no expiry, no FII/DII, no OI bhavcopy, no delivery%).
- **What transfers:** the **entire Phase 2 engine** — `StrategyBase`, walk-forward, Deflated Sharpe, SPA/FDR, vol-targeted sizing, locked holdout, execution realism — is asset-agnostic and would be reused unchanged.
- **What does NOT transfer:** all of Phase 1's NSE data layer, all of Tier D (FII/DII), every OI/CPR/expiry/VWAP-on-futures assumption.
- **Decision:** build `StrategyBase` and the data adapter **asset-class-agnostic** (a `market` field; pluggable loaders) so a future "crypto/FX extension" plugs a `ccxt` loader and reuses the engine. Bolting crypto/FX into *this* run would multiply the false-discovery surface and the NSE-specific machinery is meaningless for them. **NSE-only now; clean seam for later.**

**"Will indicator combos (Volume, RSI, VWAP, OI, Pivot, CPR, SMA/EMA/WMA, …) be tested?"** — **Yes (Tier I), and the set is expanded** to include Bollinger-band squeeze, RSI(2) Connors mean-reversion, MACD (as a baseline), ADX (as a gate), Donchian, and gap strategies. They are tested **to be falsified as much as confirmed**; each carries a "beats buy-and-hold net of costs?" flag.

## 5. The multiple-testing defense (the upgrade, summarized)

The headline ranking is **Deflated Sharpe on Holdout OOS**, with the trial count including the instrument dimension. On top of that, Phase 2/6 require:
- **White's Reality Check + Hansen's SPA** (stationary block bootstrap) to test whether the *best* strategy's performance survives data-snooping across the whole search. This is the institutional standard for "is the winner real?" and is strictly better than counting.
- **Benjamini–Hochberg (and Benjamini–Yekutieli for dependence) FDR** across the cross-sectional stock panel; report "N combos passed; ≈M expected by chance; FDR-controlled survivors = K."
- **Correlation / sector clustering** of survivors: collapse near-duplicate winners (e.g. 15 PSU banks on the same signal) into one effective discovery, and report the **effective number of independent bets**.

## 6. Full roster (v3)

Tiers: **NULL** benchmarks · **A** established · **B** regime gates/tools · **C** practitioner · **D** FII/DII crowding (index-centric) · **E** novel · **I** indicator combos · **M** meta/ensemble. *(New in v3 marked ★.)*

| ID | Name | Tier | Direction | Instruments | Note |
|----|------|------|-----------|-------------|------|
| N1 | Buy & hold | NULL | positional | index + stocks | the bar everything must beat |
| N2 | EMA crossover (+ATR filter) | NULL | both | index + stocks | baseline, not candidate |
| N3 | Supertrend | NULL | both | index + stocks | baseline |
| ★N4 | MACD crossover/histogram | NULL | both | index + stocks | ubiquitous retail baseline |
| A1 | Time-series momentum | A | positional | index + stocks | Moskowitz-Ooi-Pedersen |
| A2 | Short-horizon reversal | A | both | index + stocks | decayed; needs gate + spread penalty |
| A3 | Cointegration pairs | A | both | **index pair + ★stock pairs** | ADF/Johansen gate; stocks add breadth |
| A4 | VIX-conditioned regime | B-gate | — | index | master filter |
| A5a | Vol-risk-premium signal | A | positional | index | backtestable |
| A5b | VRP options-selling | A | positional | index | **forward-test-only** |
| ★A6 | Donchian / Turtle breakout | A | positional | index + stocks | evidenced trend-following |
| ★A7 | Cross-sectional momentum / RS | A | positional | **stocks only** | Jegadeesh-Titman; uses the whole universe |
| B1 | HMM 3-state regime | B-gate | — | index | label-switching handled |
| B2 | Hurst regime | B-gate | — | index + stocks | trend vs revert |
| B3 | Kalman trend filter | B-gate/feat | — | index + stocks | low-lag |
| ★B5 | ADX/DMI trend-strength | B-gate | — | index + stocks | suppress chop |
| B4 | GBM next-bar classifier | B | intraday | index + stocks | highest overfit risk; purged/embargoed |
| C1 | Volume Profile / POC | C | both | futures / stocks-cash | look-ahead trap |
| C2 | Anchored VWAP | C | both | futures / stocks-cash | pre-register anchor |
| C3 | SMC (OB/BoS/sweep) | C | both | index + stocks | **falsification target; expect rejection** |
| C4 | Opening Range Breakout | C | intraday | futures / stocks-cash | gate with A4/B2; fill realism |
| ★C5 | Gap-fill / gap-and-go | C | both | **stocks** (cash) | stocks gap; index rarely |
| D1 | FII futures positioning extreme | D | positional | index | contrarian crowding |
| D2 | FII cash+futures convergence | D-gate | — | index | confidence gate |
| D3 | OI buildup/unwind | D-feature | — | index + stocks | confirming feature only |
| D4 | Delivery% spike + FII cash | D-filter | — | index + stocks | filter only |
| E1 | Futures basis / roll-yield | E | positional | index + stocks(fut) | cleaner than raw OI |
| E2 | Overnight vs intraday × FII | E | both | index | as-of FII cash |
| E3 | Index dispersion regime | E-gate | — | index | sector-proxy; partially testable |
| E4 | Scheduled-event drift | E | both | index + stocks | scheduled dates only |
| I1 | EMA ribbon + RSI | I | both | index + stocks | popular |
| I2 | CPR width + pivot reaction | I | both | index + stocks | test CPR-width claim |
| I3 | Volume-spike + price/RSI | I | both | stocks-cash / index-fut | |
| I4 | OI buildup + price + MA | I | both | index/stock futures | OI grid is heuristic |
| I5 | VWAP + RSI cross | I | intraday | stocks-cash / index-fut | |
| I6 | Triple-confirm (ST+EMA+RSI) | I | both | index + stocks | **falsification: "more = better" folklore** |
| I7 | Pivot/S-R bounce + volume | I | intraday | stocks-cash / index-fut | high hindsight risk |
| ★I8 | Bollinger squeeze → breakout | I | both | index + stocks | squeeze is a real vol-regime signal |
| ★I9 | RSI(2) Connors mean-reversion | I | both | index + stocks | specific, published, testable |
| M1 | Regime-aware ensemble | M | both | index + stocks | the disciplined "all-weather" construct |

**Instrument applicability (correctness, not preference):**
- Index spot has **no volume** → all volume/VWAP/POC logic on the index uses **continuous futures**. Stocks have **real cash volume** → stock volume logic uses cash-equity OHLCV. OI logic uses **stock-futures / bhavcopy stock OI**.
- **Tier D FII/DII is INDEX-ONLY** (per-stock FII positioning is not in free EOD data). Only stock-level **OI (D3)** and **delivery% (D4)** extend to stocks.
- **Cross-sectional momentum (A7), stock pairs (A3), gap (C5)** are stock-universe constructs.

## 7. Risk doctrine (enforced by the engine; pre-registered, not optimized)

- **Risk per trade ≤ 1% of equity** (default; pre-registered, never tuned). Volatility-targeted position sizing on top.
- **Stop-loss and all exits (stop/target/time) defined before entry.** No discretionary holds.
- **Max concurrent exposure** and **max daily loss** hard caps. Optional **max trades/day** cap (pre-registered; default 2–3 intraday).
- **Reward:risk:** v2 implied a fixed 2:1. v3 treats the R:R target as a **tested hypothesis, not a law** — high-win-rate mean-reversion can have R:R < 1 and still positive expectancy. The engine reports expectancy and R-multiple; it does not force 2:1 unless a strategy's pre-registration declares it.
- **₹1-lakh interpretability block** on every result (ending equity, profit ₹, return %, CAGR, max DD ₹/%, per-year ₹ P&L, **max leverage used**, return per unit risk) shown *beside* the rank, never *as* the rank.

## 8. What to expect (set expectations before you run)

- A **large fraction of the intraday × single-stock cells will be data-thin** (< 30 trades) and correctly greyed-out/flagged — especially recently-added F&O names with short 5-min history. This is honest, not a bug.
- **Most Tier I combos and C3 (SMC) are expected to fail** the beats-null screen net of costs. Reporting the failure *is* the deliverable.
- **Intraday continuous futures are an engineering cost** (token caching + manual stitch across expiries — Kite's `continuous=1` returns only *daily* candles for expired contracts). Budget for it in Phase 1.
- The full sweep is a **multi-hour-to-multi-day** compute + fetch job. Caching and resumability are mandatory.

## 9. Execution order

0. **Phase 0** — this orientation + feasibility report.
1. **Phase 1** — data layer (indices + F&O stocks, point-in-time, corp-action-adjusted, paginated intraday, intraday futures stitching, costs).
2. **Phase 2** — methodology engine (anti-overfit scaffolding, SPA/FDR, execution realism, ₹1-lakh metrics, screener). Built before any strategy.
3. **Phase 3** — NULL + Tier A (incl. A6 Donchian, A7 cross-sectional, A3 stock pairs) + Tier B gates (incl. B5 ADX).
4. **Phase 4** — Tier C (incl. C5 gap) + D + E.
5. **Phase 5** — Tier I indicator combos (incl. I8 BB-squeeze, I9 RSI(2)).
6. **Phase 6** — full sweep, single holdout touch per instrument, all-timeframe ₹1-lakh dashboard with regime buckets, Tier M ensemble, SPA/FDR panel, limitations log, research summary, guarded meta-discovery.
7. **Phase 7** — paper-trading & forward-validation bridge (the honest step toward alerts/auto-orders).

## 10. Repo layout (target end state)

```
/data
  /raw /processed /futures
  /nse                # participant_oi (index), fii_dii_cash (index), bhavcopy/ (stock OI + delivery%)
  /stocks             # per-symbol OHLCV (cash + stock-futures continuous), corp-action-adjusted
  /pointintime        # as-of-dated revised series
  /reference          # expiry_calendar, lotsize_history, fno_membership (point-in-time), holidays, sector_map
  /panel              # aligned cross-sectional return/feature panel (for A7, A3 stock pairs)
  /chain_recorder
/strategies            # all subclass StrategyBase; tiers N/A/B/C/D/E/I/M
/engine                # runner (instrument×strategy×timeframe), walk-forward, metrics, deflated_sharpe,
                       #   reality_check (SPA), fdr, execution (fills/slippage), quick_screen
/methodology           # split, preregistration, robustness, multiple_testing, clustering
/costs.py              # date-aware cost & tax + lot-size + corp-action
/dashboard
/results               # backtests, walkforward, dashboard, screener (triage), reality_check, fdr_panel
/logs
preregistration.json
config.yaml
run_research.py
```

## 11. Non-negotiable guardrails (every phase)

1. **Locked holdout** — Train ~50% / Val ~25% / Holdout ~25%, chronological, **per instrument**. Touched once, at final ranking, never for tuning.
2. **Pre-registration** of hypotheses + bounded ranges before any backtest.
3. **Walk-forward** OOS is the headline number.
4. **Deflated Sharpe** ranks everything; trial count includes the instrument dimension. **Plus** SPA/Reality-Check + FDR + clustering for the cross-sectional panel.
5. **Point-in-time integrity** for FII/DII, expiry rules, lot sizes, cost rates, **and F&O-eligibility membership**.
6. **Net of all costs** — segment-correct STT, GST, slippage, **corp-action adjustment for stocks**, realistic fills.
7. **Rank on risk-adjusted, never on return.** ₹-return shown beside max-leverage and drawdown.
8. **Honesty over completeness.** Guardrail violations → INVALID, not ranked. Failures are valid results.

## 12. DEFINITION OF DONE (Phase 0)

- [ ] Environment confirmed (Python, Kite key+token, disk for tens of GB, network to NSE/Kite).
- [ ] Four fact sets verified-live; any drift vs this doc printed.
- [ ] Feasibility report printed: per timeframe × instrument-class realistic history depth, expected usable-trade counts, data-thin cells flagged, estimated request count + fetch wall-clock at 3 req/s.
- [ ] Scope decisions (NSE-only now; crypto/FX deferred) and the SPA/FDR/clustering defense restated in working memory.
- [ ] No strategy or data code written yet. Hand to Phase 1.
