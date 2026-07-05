# Phase 6 v4 - Full Run, Dashboard, Ranking, and Guarded Meta-Discovery

Feed after Phase 5 passes. This is the only phase that can touch the locked holdout.

## Kickoff prompt

```text
Phases 1-5 passed. Execute the full research protocol.

1. Run all non-forward-test-only strategies across instrument x strategy x timeframe on Train+Validation.
2. Tune only within pre-registered bounds.
3. Apply execution realism, costs, risk manager, robustness, and Monte Carlo stress.
4. Compute Deflated Sharpe, SPA/Reality Check, FDR, and clustering.
5. Select candidate configs without holdout.
6. Touch each instrument's holdout exactly once for final ranking.
7. Build the dashboard and ranked CSV.
8. Promote, demote, retire, or forward-watch every strategy.
9. Build Tier M only from validated sub-strategies.
10. Run guarded meta-discovery only after the main report, with new hypotheses marked forward-test-only.

Any guardrail violation is INVALID and not ranked.
```

## Full sweep

Run:

```text
instrument x strategy x timeframe x variant
```

Across:

- NIFTY.
- BANKNIFTY.
- Index futures.
- Full point-in-time FNO stock universe.
- Stock futures where required.

Forward-test-only items stay separate.

## Candidate selection before holdout

Before touching holdout, rank on Train+Validation walk-forward OOS using:

- Deflated Sharpe.
- Calmar.
- Max drawdown.
- Expectancy.
- Trade count.
- Cost sensitivity.
- Parameter stability.
- Beats-null flags.
- SPA/Reality Check preliminary set.
- FDR/clustering for stocks.

The holdout is not used for discovery.

## Holdout evaluation

For selected configs:

- Evaluate exactly once.
- No tuning.
- No parameter edits.
- Log holdout access.
- If holdout collapses, report collapse and demote.

## Dashboard requirements

Single HTML dashboard plus CSV/Parquet outputs:

- Strategy rank by Deflated Sharpe on holdout OOS.
- All timeframes side by side.
- Index vs stock filter.
- Per-stock drilldown.
- Sector aggregation.
- Intraday vs positional flags.
- "Good in both" flag.
- INR 1,00,000 ending equity, profit, return percent, max drawdown INR/percent.
- Max leverage and margin used.
- Calmar, Sortino, Sharpe, Deflated Sharpe.
- Win rate, profit factor, expectancy, R-multiple.
- Trade count and data-thin labels.
- Cost drag and slippage sensitivity.
- Per-regime buckets.
- Equity curve and drawdown chart.
- SPA p-value for top strategies.
- FDR panel for stock strategies.
- Correlation/sector cluster view.
- Invalid and retired strategy cemetery with reasons.
- Forward-test-only section.
- Data-quality and limitations panel.

## Promotion/demotion output

Every strategy receives:

- Validation state.
- Reason.
- Best instrument/timeframe.
- Worst regime.
- Main risk.
- Whether it is eligible for Phase 7.

States:

- Validated candidate.
- Forward-watch.
- Exploratory/data-thin.
- Retired.
- Invalid.
- Forward-only.

## Tier M ensemble

M1 can only use sub-strategies that already passed:

- Holdout.
- Costs.
- Robustness.
- Multiple-testing gates.

M1 allocation rule must be pre-registered and counted as a new strategy. It can rotate by regime and recent validated performance, but it must sit small or flat when no edge is active.

## Guarded meta-discovery

Claude/Codex may propose new strategies only under these rules:

1. Hypothesis must be mechanism-based, not "best-looking chart."
2. Pre-register before testing.
3. Count in trial penalty.
4. Never use spent holdout for discovery.
5. Mark as forward-test-only until validated on future unseen data.
6. Report how many hypotheses were generated and rejected.

Examples of allowed meta-discovery:

- A footprint feature works only after high-dispersion days: propose a gate.
- ORB fails in low breadth but works in high breadth: propose breadth-gated ORB.
- Stock momentum works only outside earnings weeks: propose event filter.

Examples of banned meta-discovery:

- Change RSI threshold repeatedly until profit appears.
- Add filters after seeing holdout.
- Promote a single lucky stock/timeframe without FDR support.

## Research summary

Must state:

- No live-trading recommendation.
- Backtest rank is a filter, not a guarantee.
- Forward validation is required before alerts.
- Auto-orders are a separate later project.
- Top candidates include caveats: drawdown, regime dependence, cost sensitivity, sample size, SPA/FDR, and execution assumptions.

## Definition of Done

- Full Train+Validation sweep complete.
- Complete trial count computed.
- Deflated Sharpe, SPA/Reality Check, FDR, and clustering complete.
- Holdout touched exactly once per selected instrument/config.
- Dashboard and ranked CSV written.
- INR 1,00,000 metrics shown.
- Strategy cemetery included.
- Tier M built only from validated strategies.
- Limitations log and data-quality log written.
- Meta-discovery, if run, is forward-test-only and fully disclosed.

