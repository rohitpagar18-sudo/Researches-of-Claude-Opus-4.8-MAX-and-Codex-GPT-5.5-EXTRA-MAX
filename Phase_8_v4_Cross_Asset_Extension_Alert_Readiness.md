# Phase 8 v4 - Cross-Asset Extension and Alert Readiness

This is optional and comes after Phase 7. It answers how the system can later support Bitcoin, Forex, alerts, and eventually separate order-placement projects.

## Kickoff prompt

```text
Phase 7 completed and the human selected strategies worth extending.

Build only the extension plan and adapters needed for the next research market or alert prototype.

1. Do not mix NSE results with crypto/FX in one statistical ranking.
2. Create separate market adapters, cost models, calendars, and execution assumptions.
3. Reuse StrategyBase, risk manager, walk-forward, metrics, dashboard, and validation framework.
4. For alerts, build signal-only dry-run outputs first.
5. Do not place live orders in this phase.
```

## What transfers to Bitcoin and Forex

Reusable:

- StrategyBase.
- Risk manager.
- Walk-forward.
- Locked holdout.
- Deflated Sharpe.
- SPA/Reality Check.
- FDR/clustering where applicable.
- Dashboard.
- Paper harness.
- Promotion/demotion rules.

Must be rebuilt:

- Data loader.
- Cost model.
- Trading calendar.
- Funding model.
- Slippage/spread model.
- Instrument metadata.
- Session definitions.
- Market-specific microstructure features.

## Crypto adapter

Likely data sources:

- Exchange REST/WebSocket APIs.
- CCXT-style normalized feeds.
- Perpetual futures funding data.
- Order book snapshots.

Differences vs NSE:

- 24/7 trading.
- No overnight gap in the same sense.
- Maker/taker fees.
- Perpetual funding.
- Exchange-specific liquidity and outages.
- Different leverage/margin rules.

Strategies likely portable:

- Time-series momentum.
- Donchian breakout.
- Short-horizon reversal.
- VWAP/AVWAP.
- Volume/RVOL footprint.
- Bollinger/Keltner squeeze.
- Regime-aware ensemble.

Strategies not portable as-is:

- FII/DII flows.
- NSE expiry rules.
- NSE delivery percentage.
- India VIX rules.
- Stock FNO membership.

## Forex adapter

Likely data sources:

- Broker feed such as OANDA or another FX broker.
- Spread history where available.
- Economic calendar.

Differences vs NSE:

- OTC market.
- Broker-specific spreads.
- Rollovers/swaps.
- 24x5 trading.
- No centralized volume for spot FX.

Strategies likely portable:

- Time-series momentum.
- Breakout.
- Mean-reversion with spread realism.
- Regime filters.
- Event windows for scheduled macro releases.

Strategies not portable as-is:

- Exchange volume profile unless using futures/proxy.
- Delivery percentage.
- FII/DII.
- Stock cross-sectional momentum unless using a currency basket.

## Alert readiness

Only after a Phase 7 PASS or long DEGRADED watchlist:

Build per strategy:

- Frozen config.
- Signal generator.
- Data freshness check.
- Risk check.
- Duplicate alert guard.
- Human-readable alert payload.
- Dashboard status.
- Audit log.
- Kill switch.

Alert payload:

```text
strategy_id
instrument
timeframe
signal_time
direction
entry_zone
stop
target_or_exit_rule
risk_percent
position_size_suggestion
reason_codes
regime_state
data_quality_status
config_hash
```

No orders yet.

## Auto-order placement is a separate project

Order placement requires a new safety design:

- Broker permissions.
- Live margin checks.
- Order type mapping.
- Partial fill handling.
- Rejection handling.
- Kill switch.
- Max position limits.
- Max loss limits.
- Monitoring and alerting.
- Manual override.
- Disaster recovery.

Do not combine research and auto-trading in one phase.

## Definition of Done

- Market adapter plan written.
- Cost/execution differences documented.
- Portable vs non-portable strategies mapped.
- Alert-only architecture drafted.
- No live orders implemented.
- Any future cross-asset study has its own preregistration and trial count.

