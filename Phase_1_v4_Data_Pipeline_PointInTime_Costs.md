# Phase 1 v4 - Data Pipeline, Point-in-Time Stores, and Costs

Feed after Phase 0 passes.

## Kickoff prompt

```text
Phase 0 passed. Build the data layer for the NSE research system.

Read this phase completely, then build:

1. Repo skeleton and config.yaml.
2. Date-aware costs.py for brokerage, STT/CTT, exchange charges, GST, SEBI fees, stamp duty, slippage, margin, and lot-size history.
3. Kite instrument master cache and token registry.
4. Historical OHLCV loaders for index, futures, options, stock cash, and stock futures.
5. Runtime API probes to discover Kite historical fetch-window limits and throttling behavior; do not assume hard-coded limits.
6. Continuous futures: day candles using Kite continuous support where available; intraday stitched from cached contract tokens.
7. FNO stock universe builder with point-in-time add/remove dates.
8. Corporate-action adjustment for stock daily and intraday data.
9. Cross-sectional stock panel with eligibility masks.
10. NSE EOD scrapers for participant OI, FII/DII cash, FNO bhavcopy, delivery data, holidays, expiry calendar, and lot-size references.
11. Option-chain and market-depth recorder for forward-only studies.
12. Resumable, rate-aware fetch orchestrator with cache-first behavior.

Print the data inventory and verify the Definition of Done.
```

## Required repo layout

```text
/data
  /raw
  /processed
  /futures
  /stocks
  /nse
  /pointintime
  /reference
  /panel
  /chain_recorder
  /depth_recorder
/strategies
/engine
/methodology
/dashboard
/results
/logs
config.yaml
costs.py
preregistration.json
run_research.py
```

## Data principles

- Cache once, reuse forever.
- Every dataset has an as-of date, source, fetch timestamp, and checksum.
- A backtest can only use information available at that historical time.
- Missing data is logged and surfaced; never silently forward-fill a signal-critical field.
- Point-in-time FNO eligibility is mandatory.
- Corporate actions are mandatory for stock history.

## Kite facts and runtime probes

Kite's official documentation states that historical candles span several years and that continuous data works for NFO/MCX futures day candles of expired contracts when using a live contract token. It also states that instrument tokens change by expiry and that expired tokens must be cached if needed later.

Therefore Phase 1 must:

- Download the instrument master daily and cache it.
- Store exchange plus tradingsymbol as the stable key; do not trust token permanence.
- Probe historical API request windows by interval at runtime.
- Back off on throttling and log real observed rate.
- Fetch with pagination and resume from checkpoints.
- Use `oi=1` where futures/options OI is required.

Do not treat any unofficial request-window number as permanent.

## Instrument coverage

### Indices

- NIFTY 50 spot series for price reference.
- BANKNIFTY spot series for price reference.
- INDIA VIX for regime features.

### Futures

- NIFTY front-month continuous futures.
- BANKNIFTY front-month continuous futures.
- Stock futures for all eligible FNO stocks where data exists.
- Unadjusted per-contract futures for basis and OI work.
- Adjusted continuous futures for indicators and backtesting.

### Stocks

- Cash-equity OHLCV for all point-in-time FNO-eligible stocks.
- Daily and intraday where available.
- Corporate-action-adjusted price and volume.
- Delisted/removed names preserved if data can be sourced.

### Options

- Historical option candles where Kite provides them.
- Live option-chain recorder from Phase 1 onward.
- Options strategies without historical premium data are forward-test-only.

## FNO stock universe

Build:

```text
/data/reference/fno_membership.parquet
columns: symbol, name, sector, add_date, remove_date, source, source_timestamp
```

Rules:

- Use full universe by default.
- Current count is verified live.
- Historical membership must be point-in-time.
- A stock is eligible only between add_date and remove_date.
- Random subsets are allowed only for smoke tests.
- Compute-limited research subsets must be stratified by sector, liquidity, and volatility and must be pre-registered.

## Cross-sectional panel

Build date x symbol matrices:

- Adjusted close.
- Returns.
- Volatility.
- Momentum lookbacks.
- Liquidity and turnover.
- Gap size.
- Sector.
- Eligibility mask.
- Delivery percentage where available.
- Stock futures OI where available.
- OBV, ADL, CMF, RVOL, MFI, and volume contraction metrics for Tier F/I.

No feature at date t may use data after t.

## Cost module

`costs.py` must expose:

```python
net_cost(segment, side, price, qty, trade_date, order_type, liquidity_bucket, order_size)
estimate_slippage(instrument, side, qty, price, timestamp, strategy_class)
lot_size(instrument, trade_date)
margin_required(instrument, segment, qty, price, trade_date)
```

Zerodha's public charges page must be checked at runtime for current values. Store a versioned table for historical backtests.

Minimum cost components:

- Brokerage.
- STT/CTT.
- Exchange transaction charges.
- GST.
- SEBI fees.
- Stamp duty.
- Slippage.
- Bid/ask spread.
- Impact cost by liquidity bucket.
- Margin and leverage assumptions.

## Corporate actions

For stocks:

- Adjust OHLC and volume for splits and bonuses.
- Track dividends separately; do not fake intraday dividend adjustment without a documented method.
- Verify on at least one known split.
- Keep raw and adjusted data.

## Expiry and lot-size calendar

Build point-in-time tables:

- Trading holidays.
- Index and stock expiry dates.
- Weekly vs monthly rules by historical date.
- NIFTY/BANKNIFTY lot-size history.
- Stock futures lot-size history from daily contract files.

Every expiry-sensitive strategy must call this calendar.

## Data-quality log

Every fetch writes:

- Source.
- Instrument.
- Date range.
- Interval.
- Rows fetched.
- Missing bars.
- Duplicates.
- Corporate actions applied.
- First and last available date.
- Cache path.
- Warnings.

## Definition of Done

- Repo skeleton imports cleanly.
- Kite instrument master cached and token registry created.
- API probe prints real observed request-window and throttling behavior.
- Sample data fetched for one index, one index future, one option, three stocks, and three stock futures.
- >1 year of 5m data assembled for at least one sample where available.
- Daily continuous futures built using continuous support where possible.
- Intraday continuous futures stitched across at least two expiries if cached tokens/data are available; otherwise limitation is printed.
- FNO membership table returns correct as-of eligibility.
- Corporate-action adjustment verified.
- Costs module prints sample calculations for equity delivery, equity intraday, futures, and options.
- NSE EOD data fetchers cache and resume.
- Panel built with eligibility masks.
- Option-chain/depth recorder writes one snapshot.
- Data inventory printed.

