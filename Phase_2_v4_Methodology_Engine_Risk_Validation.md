# Phase 2 v4 - Methodology Engine, Risk Controls, and Validation

Feed after Phase 1 passes. No real strategy should run before this exists.

## Kickoff prompt

```text
Phase 1 passed. Build the methodology engine before strategy implementation.

Build:

1. StrategyBase with a uniform asset-class-aware interface.
2. Chronological train/validation/holdout split per instrument; panel split by date.
3. Holdout access guard that raises during tuning.
4. Pre-registration writer.
5. Walk-forward engine with purge/embargo support.
6. Bounded tuner that cannot escape pre-registered ranges.
7. Execution realism: next-bar fills, spread crossing, limit-fill probability, slippage, impact, lot rounding, and margin.
8. Risk manager: 1% default risk, <=2% hard cap, stop before entry, max trades/day, max daily loss, max concurrent exposure.
9. Metrics: risk-adjusted, INR 1,00,000 block, trade expectancy, R-multiple, drawdown, per-regime, per-year.
10. Multiple-testing: Deflated Sharpe, White Reality Check, Hansen SPA, BH/BY FDR, clustering.
11. Monte Carlo and 100-trade simulator.
12. Promotion/demotion engine.
13. quick_screen triage that cannot touch holdout.

Demonstrate with a dummy strategy on one index and one stock.
```

## StrategyBase

```python
class StrategyBase:
    id: str
    name: str
    tier: str
    validation_state: str
    novelty: str
    direction: str
    instruments: str
    market: str = "NSE"
    timeframes: list
    data_kind: str
    cross_sectional: bool = False
    forward_test_only: bool = False
    params: dict
    param_bounds: dict
    hypothesis: str
    structural_rationale: str

    def generate_signals(self, data, context):
        raise NotImplementedError
```

The runner, not individual strategies, enforces sizing, stops, fills, costs, and guardrails.

## Splits

- Train: about 50%.
- Validation: about 25%.
- Holdout: about 25%.
- Chronological per instrument.
- Cross-sectional strategies split by date across the whole panel.
- Holdout can be touched only in Phase 6 final ranking.

## Pre-registration

Before a strategy runs, write:

- ID and name.
- Hypothesis.
- Instruments and timeframes.
- Data kind.
- Entry, exit, stop, target, and time-stop rules.
- Parameter bounds.
- Regime gates.
- Risk settings.
- Expected failure modes.
- Trial-count contribution.

No post-result widening of parameter bounds.

## Risk manager

Every proposed trade must produce:

- Entry price assumption.
- Stop price.
- Stop distance.
- Risk in INR.
- Risk as percent of equity.
- Quantity after lot rounding.
- Margin required.
- Expected slippage and cost.
- Target or exit rule.
- Planned reward:risk where applicable.
- Reason to skip if minimum lot causes excess risk.

Hard defaults:

- Risk per trade: 1%.
- Absolute max risk per trade: 2%.
- Max intraday trades: 2 or 3 unless pre-registered otherwise.
- Max daily loss: pre-registered, default 2% to 3%.
- Max weekly loss: pre-registered.
- Max concurrent exposure by instrument, sector, and strategy.
- Circuit-limit and illiquidity guard.

## Execution realism

Default fill model:

- Signals generated on closed bars.
- Entry at next bar open or worse.
- Stop and target simulated with intrabar ordering assumptions that are conservative.
- Mean-reversion pays spread-crossing penalty.
- Breakouts apply missed-fill or trade-through probability.
- Liquidity and order-size slippage scales by instrument.
- Stock FNO mid/small names receive higher impact assumptions.
- Results shown at 1x and 2x slippage.

## Metrics

Per strategy x instrument x timeframe:

- Total return.
- CAGR.
- Max drawdown percent and INR.
- Drawdown duration.
- Sharpe, Sortino, Calmar.
- Deflated Sharpe.
- Win rate.
- Average win/loss.
- Profit factor.
- Expectancy.
- Average R-multiple.
- Trade count.
- Turnover.
- Cost drag.
- Slippage drag.
- Max leverage used.
- Max margin used.
- Per-year returns.
- Per-regime returns.
- IS vs OOS gap.
- Parameter sensitivity.
- 1x vs 2x slippage survival.
- INR 1,00,000 normalized ending equity and profit.

Data-thin labels:

- <30 trades: data-thin.
- 30-99 trades: exploratory.
- >=100 trades: normal for intraday and short-horizon strategies.
- Positional strategies may use fewer trades but require longer calendar coverage and stronger forward validation.

## Regime buckets

At minimum:

- Bull trend.
- Bear trend.
- Sideways/chop.
- High volatility.
- Low volatility.
- Expiry week.
- Event window.

For stocks:

- High dispersion vs low dispersion.
- Sector leadership vs weak sector.
- High liquidity vs low liquidity.

## Multiple-testing defense

Apply all:

- Deflated Sharpe with effective trial count.
- White Reality Check for data snooping.
- Hansen SPA for superior predictive ability.
- BH and BY FDR for stock-panel p-values.
- Correlation and sector clustering to estimate independent discoveries.

The trial count includes:

- Instruments.
- Timeframes.
- Strategy variants.
- Parameter trials.
- Pair scans.
- Meta-discovered hypotheses.

## 100-trade and Monte Carlo stress

For each candidate:

- Simulate 100-trade sequences using observed R-multiple distribution.
- Bootstrap 500+ paths of trades.
- Estimate risk of ruin.
- Estimate max losing streak.
- Estimate probability of 10%, 20%, and 30% drawdown.
- Stress with doubled costs and worse fills.
- Stress with regime-specific trade subsets.

This answers whether the edge survives normal bad luck.

## quick_screen

Allowed only for engineering triage:

- No holdout.
- No ranking.
- No final selection.
- Coarse/default params only.
- Full costs still applied.
- Output stamped "UNVALIDATED TRIAGE".

## Promotion/demotion engine

Promotion requires:

- Beats null baseline.
- Positive holdout OOS.
- Acceptable max drawdown.
- Deflated Sharpe threshold passed.
- SPA/FDR acceptable where applicable.
- Robust to 2x slippage.
- Parameter sensitivity not brittle.
- Trade count sufficient or explicitly marked data-thin.

Demotion/retirement triggers:

- Negative net expectancy after costs.
- Holdout collapse.
- Look-ahead or survivorship issue.
- Overfit parameter island.
- Cost fragility.
- Correlated clone with no independent evidence.

## Definition of Done

- Dummy strategy runs end-to-end on one index and one stock.
- Holdout guard raises during tuning.
- Pre-registration file written.
- Walk-forward report produced.
- Costs and execution realism applied.
- Risk manager sizes/skips trades correctly.
- Metrics include INR 1,00,000 block.
- Multiple-testing modules run on toy data.
- Monte Carlo report generated.
- quick_screen cannot access holdout.
- Promotion/demotion labels generated.

