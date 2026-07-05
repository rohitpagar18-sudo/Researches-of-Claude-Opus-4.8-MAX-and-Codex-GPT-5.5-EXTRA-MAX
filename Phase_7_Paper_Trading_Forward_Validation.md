# PHASE 7 — Paper-Trading & Forward-Validation Bridge (v3 — NEW)

> Feed after Phase 6 passes. This is the honest step between *"ranked in a backtest"* and *"worth an alert."* **Paper only — no live orders, no real capital in this pack.**

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phase 6 is complete: the dashboard is built, the holdout is spent, and a small set of top
strategies (with where-it-works clear: instrument, timeframe, regime) has been selected. Now
build the forward-validation bridge per this Phase 7 doc. Build, in order:
1. forward/config: read the selected strategies + their FROZEN params/config hashes from Phase 6.
   No re-tuning. No parameter changes. The config hash must match the dashboard's.
2. A paper-trading harness that runs the selected strategies forward on data generated AFTER the
   backtest cutoff only (live/recorded feed via the Phase 1 chain recorder + fresh OHLCV pulls).
   It simulates fills using the SAME Phase 2 execution-realism model (next-bar fill, spread cross,
   fill-probability haircut, liquidity-scaled slippage) and the SAME costs.py.
3. A forward metrics + reconciliation module: track the live-forward metric set per strategy and
   compare it head-to-head against that strategy's backtest/holdout numbers. Compute the
   degradation ratio (forward Sharpe / backtest Sharpe), expectancy decay, hit-rate drift, and
   drawdown vs backtest-max-DD. Reconcile MODELED fills/slippage/costs vs the harness's realized
   fills — this validates (or falsifies) the Phase 2 execution assumptions.
4. The forward-test-only strategies that could never be historically backtested (A5b VRP options
   selling; any intraday option-chain variants) get their FIRST real evaluation here, on
   accumulated recorder data, in a clearly separate section.
5. A decision report with explicit PASS / DEGRADED / FAIL gates (defined below) per strategy, and
   a written go/no-go for each — framed as INPUT to the human's decision, never an instruction.
Honest framing throughout: a few weeks of forward data is a SMALL sample; state confidence limits;
do not over-conclude. NO live-trading recommendation. NO order placement. Verify Definition of Done.
```

---

## 1. Why this phase exists (the overfitting check that the holdout cannot fully give you)

The locked holdout in Phase 6 is the strongest *in-sample* defense available, but it is still in-sample to the **research process**. By the time a strategy reaches the top of the dashboard, it has survived a selection tournament across tens of thousands of cells; the holdout was touched once, but the *act of choosing* which configs to send to the holdout used information from Train+Val. The only data that is genuinely, structurally unseen is **data that did not exist when the backtest was run.** Phase 7 is where the selected strategies meet that data.

This is the single most informative honesty test in the whole pack. A strategy whose forward performance roughly tracks its backtest has earned real credibility. A strategy whose forward performance collapses was an overfit — and it is far cheaper to discover that on paper here than with capital later. Expect meaningful decay on most candidates; that is the normal, healthy outcome of an honest pipeline, not a failure of the work.

## 2. Prerequisites (Phase 6)

A finished dashboard; a selected short-list of strategies with **frozen** params and config hashes; the Phase 1 chain recorder accumulating live snapshots; the Phase 2 execution-realism + costs modules.

## 3. Freeze, then forward (no re-tuning — this is the whole point)

- Read the selected strategies and their exact params from the Phase 6 results. **Re-tuning, re-fitting, or "small adjustments" here invalidate the test** — they reintroduce the look-ahead Phase 7 exists to detect. Assert the config hash matches the dashboard's; mismatch → raise.
- Walk-forward *re-fitting* (a strategy's own rolling re-estimation, declared in Phase 2) is allowed **only** where it was part of the strategy's pre-registered design and uses solely the live forward window as it unfolds — never future bars, never a peek back into Train/Val to "refresh."
- The forward window starts at the backtest cutoff date and runs forward in wall-clock time. Recommended **minimum 4–8 weeks** for intraday strategies (enough sessions for a non-trivial trade count) and **one quarter or more** for positional/daily strategies. Sub-15-minute strategies need the most calendar time to accumulate a usable sample. State the realized trade count with every forward result.

## 4. The paper-trading harness

- Runs each selected strategy forward on **post-cutoff data only**: fresh OHLCV pulls (indices, futures, F&O stocks) + the chain recorder's accumulated option snapshots. Nothing from the historical stores feeds a forward signal except the rolling lookback each strategy legitimately needs (and that lookback must itself be post-cutoff once enough forward bars exist; until then the strategy is "warming up" and its signals are flagged provisional).
- Simulated fills use the **identical** Phase 2 execution-realism model — next-bar fill, spread-crossing penalty for mean-reversion entries, breakout fill-probability haircut, liquidity/size-scaled slippage — and the **identical** `costs.py` (segment-correct STT incl. the post-1-Apr-2026 rates, GST, brokerage, slippage). Using a gentler fill model forward than backward would manufacture a fake "it still works."
- No broker order is placed. This is a simulation against a live feed, not a connection to a trading account. (Auto-order placement is a separate later project — see §9.)

## 5. Forward metrics + the reconciliation that validates your cost model

Track the full metric set per strategy on the forward window (Sharpe, Sortino, Calmar, expectancy, hit rate, profit factor, max DD, ₹1-lakh normalized P&L, per-regime split where the forward window spans regimes), then compare **head-to-head against the backtest/holdout numbers**:

- **Degradation ratio** = forward Sharpe ÷ backtest-holdout Sharpe. Also report expectancy decay, hit-rate drift, and forward-DD vs backtest-max-DD (a forward drawdown that already exceeds the backtest's worst is a loud warning).
- **Execution reconciliation** — compare the harness's *realized* fills, slippage, and cost drag against what the Phase 2 model *predicted* for the same trades. If realized slippage runs materially worse than modeled, the backtest's costs were optimistic and every dashboard number for similar strategies should be read down. This step quietly audits the most error-prone assumption in the whole system.
- **Regime context** — note which regime(s) the forward window actually covered. A strategy that "passed forward" only in a quiet trending stretch has not been tested against the chop or the vol-spike it will eventually meet; say so explicitly.

## 6. Forward-test-only strategies get their first real test here

A5b (VRP options-selling) and any intraday option-chain variants could never be historically backtested — free data cannot backfill expired option premiums. The chain recorder has been accumulating snapshots since Phase 1; **Phase 7 is the first point at which these strategies have real data to run on.** Evaluate them in a clearly separate section, with their characteristically short samples flagged, and apply hard defined-risk caps (tail-heavy structures only). They are never ranked against the historically-backtested strategies — they are simply now *evaluable* where before they were unverifiable.

## 7. Decision gates (input to the human, never an instruction)

For each strategy, the report assigns one of three states and a one-line rationale:

- **PASS** — forward metrics broadly track the backtest (degradation ratio within a pre-registered band, e.g. ≥ ~0.6), forward drawdown inside backtest expectations, execution reconciliation clean, trade count non-trivial. *Candidate to carry into the separate alerts project — pending the human's judgment.*
- **DEGRADED** — directionally alive but materially weaker (degradation ratio below band, or forward DD near/over backtest max, or realized costs notably worse than modeled). *Inconclusive; needs a longer forward window before any further step. Not promoted.*
- **FAIL** — forward performance collapses, sign flips, or drawdown breaches a pre-registered hard limit. *Reject. The backtest rank was an artifact; do not revisit it.*

These gates are **decision inputs, not orders.** The system does not "approve" a strategy for trading. It hands the human a clear, honest forward-vs-backtest comparison and a recommended state, and the human decides what (if anything) happens next.

## 8. DEFINITION OF DONE

- [ ] Selected strategies + frozen params/config hashes loaded from Phase 6; hash-match asserted (mismatch raises). No re-tuning path exists in this phase.
- [ ] Paper harness runs the short-list forward on post-cutoff data only; warm-up signals flagged provisional until the lookback is fully post-cutoff.
- [ ] Fills/costs use the identical Phase 2 execution-realism model and `costs.py` (verified same code path as the backtest).
- [ ] Forward metric set computed per strategy with realized trade count shown; degradation ratio, expectancy decay, hit-rate drift, forward-DD-vs-backtest reported.
- [ ] Execution reconciliation (modeled vs realized fills/slippage/cost drag) produced; discrepancies surfaced.
- [ ] Forward-test-only strategies (A5b, chain variants) evaluated on recorder data in a separate labeled section with sample-size flags and defined-risk caps.
- [ ] Per-strategy PASS / DEGRADED / FAIL state + rationale written, explicitly framed as input to the human's decision.
- [ ] Small-sample confidence limits stated; **no live-trading recommendation; no order placement anywhere in this phase.**

## 9. After this pack (the seam to alerts → auto-orders)

Phase 7 closes the research pack. What it produces is a short list of strategies that have (a) survived the full anti-overfit pipeline, (b) cleared a locked holdout, and (c) not fallen apart on genuinely unseen forward data — each with its instrument, timeframe, regime, and honest caveats attached.

A **separate later project** takes a strategy the human selects from that list and builds, per strategy, its own: live alert system → (optionally, much later) automated order placement, with its own risk controls, monitoring, kill-switch, and capital limits. That project is where live broker integration, real position sizing against real capital, and order-management safety belong — none of which exists in this research pack. Forward validation here is the filter that decides what is even *worth* building an alert for. **Backtest rank is a filter, not a guarantee; forward survival is a stronger filter; neither is a promise of future profit, and selection remains the human's decision.**
