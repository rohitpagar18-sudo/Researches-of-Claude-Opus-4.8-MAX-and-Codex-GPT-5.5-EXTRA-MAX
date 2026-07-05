# Phase 7 v4 - Paper Trading and Forward Validation

Feed after Phase 6 dashboard is complete and a small shortlist is selected.

## Kickoff prompt

```text
Phase 6 passed and a shortlist was selected. Build the forward-validation bridge.

1. Load selected strategies with frozen parameters and config hashes.
2. Assert hash match with Phase 6 dashboard.
3. Run a paper harness only on post-backtest-cutoff data.
4. Use the exact same costs.py, risk manager, and execution model.
5. Reconcile modeled fills/slippage with observed live/paper fills.
6. Compute forward metrics and degradation ratios.
7. Evaluate forward-test-only strategies in a separate section.
8. Label each strategy PASS, DEGRADED, or FAIL as input to the human decision.

No retuning. No live orders. No trading recommendation.
```

## Purpose

The holdout is strong but still part of the research process. Forward data did not exist when the backtest ran. Phase 7 tests whether shortlisted strategies survive genuinely new data.

## Freeze rules

- Parameters frozen.
- Code version frozen.
- Config hash must match dashboard.
- No threshold edits.
- No extra filters.
- No retuning.

If the hash mismatches, raise and stop.

## Forward window

Recommended minimum:

- Intraday: 4-8 weeks minimum, but prefer >=100 trades before strong conclusions.
- Daily/positional: at least one quarter; preferably multiple regimes.
- Options forward-only: enough chain snapshots for meaningful sample, usually slow.

Small samples must be labeled inconclusive.

## Paper harness

Inputs:

- Fresh OHLCV after cutoff.
- Chain recorder snapshots.
- Depth recorder snapshots if used.
- Current costs and lot sizes.

Outputs:

- Paper trades.
- Modeled fill.
- Observed next-bar/fill proxy.
- Slippage estimate.
- Costs.
- Risk used.
- Stop/target behavior.
- Signal reason.

No broker order placement.

## Forward metrics

Per strategy:

- Forward return.
- INR 1,00,000 normalized P&L.
- Sharpe/Sortino/Calmar where sample allows.
- Win rate.
- Profit factor.
- Expectancy.
- R-multiple.
- Max drawdown.
- Trade count.
- Cost drag.
- Slippage.
- Regime context.

Compare to Phase 6:

- Forward Sharpe / holdout Sharpe.
- Forward expectancy / holdout expectancy.
- Hit-rate drift.
- Forward drawdown vs holdout max drawdown.
- Modeled vs realized slippage.
- Modeled vs realized missed fills.

## PASS / DEGRADED / FAIL

### PASS

- Degradation ratio acceptable.
- Drawdown inside expected range.
- Execution reconciliation clean.
- Trade count meaningful.
- No rule break.

### DEGRADED

- Edge still directionally alive, but weaker.
- Slippage or drawdown worse than expected.
- Trade count too low or regime too narrow.
- Needs more forward time.

### FAIL

- Performance sign flips.
- Drawdown breaches hard limit.
- Realized costs invalidate edge.
- Signals cannot be filled realistically.
- Rule or data assumptions break.

## Forward-test-only strategies

Evaluate separately:

- A5b defined-risk options premium structures.
- F5 order-book/depth strategies.
- Any option-chain intraday strategy.

They do not compete with historically backtested strategies until enough forward data exists.

## Definition of Done

- Frozen configs loaded and hashes verified.
- Paper harness runs post-cutoff data only.
- Same costs/risk/fill code path used as backtest.
- Forward metrics and degradation report written.
- Execution reconciliation written.
- PASS/DEGRADED/FAIL labels assigned with rationale.
- Small-sample warnings included.
- No live orders placed.

