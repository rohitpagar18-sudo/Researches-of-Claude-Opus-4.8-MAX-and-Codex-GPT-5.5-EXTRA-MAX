# PHASE 1 — Data Pipeline, Stores & Cost Module (v3)

> Feed after Phase 0. Do not proceed to Phase 2 until the Definition of Done passes.
> **All hard facts below were verified live (June 2026). Re-verify at build time and print any drift.**

---

## KICKOFF PROMPT (copy-paste to Claude Code)

```
Phase 0 (orientation + feasibility report) is complete. Build the data layer for an NSE strategy-
research system covering NIFTY 50, BANKNIFTY, and the F&O-eligible STOCK universe. Research only.
Read this entire Phase 1 doc, then build in order:
1. config.yaml + repo skeleton (Phase 0 repo layout).
2. costs.py — DATE-AWARE single source of truth for STT, brokerage, exchange charges, GST, stamp duty,
   SEBI fees, lot sizes, slippage, AND corporate-action adjustment for stocks. VERIFY rates live
   (NSE circulars + Zerodha charges page) before trusting the tables here; print any drift.
3. yfinance loaders: ^NSEI, ^NSEBANK, ^INDIAVIX, per-stock daily (verify tickers live).
4. Kite loaders with PAGINATION: index, FUTURES, options, per-stock cash + stock-futures OHLCV.
   The Kite per-request caps are NOT total availability — paginate to fetch multi-year history.
5. Roll-adjusted CONTINUOUS FUTURES (daily AND intraday) for NIFTY/BANKNIFTY and per F&O stock.
   NOTE: Kite continuous=1 returns DAILY candles for expired contracts only; intraday continuity must be
   stitched manually from cached per-contract minute data — implement the token cache + stitcher.
6. F&O STOCK UNIVERSE builder with a POINT-IN-TIME membership table (add/remove dates). No survivorship.
7. Aligned cross-sectional PANEL (/data/panel): date x symbol matrix of returns + features, eligibility-
   masked, for cross-sectional momentum (A7) and stock-pair search (A3).
8. NSE EOD scraper: participant OI (FII/DII/Pro/Client — INDEX only), FII/DII provisional cash (index),
   F&O bhavcopy (per-symbol strike OI, stock OI, delivery%). Browser headers, cookie priming, backoff,
   aggressive cache.
9. POINT-IN-TIME store for provisional/revised series (known-date, not described-date).
10. Point-in-time EXPIRY CALENDAR + LOT-SIZE history + SECTOR map (tables in this doc).
11. CORPORATE-ACTION handling (splits/bonus/dividends) for daily AND intraday + delisting/survivorship.
12. Option-chain RECORDER (daily snapshots forward).
13. Data-quality log + a RATE-LIMIT-AWARE, RESUMABLE fetch orchestrator (3 req/s historical; checkpoint
    progress so a re-run resumes from cache, never re-hits fetched ranges).
Verify compile + Definition of Done. Print a data inventory (index + stock coverage, realistic depths).
Work autonomously; ask only if a source is fundamentally unavailable.
```

---

## 1. Objective

Stand up every source — **indices and the F&O stock universe** — with point-in-time integrity, a single date-aware cost module, and a cross-sectional panel, so Phases 2–7 assume clean, leakage-free, cost-aware, corp-action-adjusted inputs.

## 2. Data sources

| Source | Data | Granularity | Realistic history | Cost | Notes |
|--------|------|-------------|-------------------|------|-------|
| yfinance | NIFTY, BANKNIFTY, INDIA VIX | Daily | ~15–20 yr | Free | Verify `^NSEI`,`^NSEBANK`,`^INDIAVIX`; cross-check vs Kite. |
| yfinance | Per-stock (e.g. RELIANCE.NS) | Daily | Long | Free | Adjusted close for corp actions; verify symbol mapping. |
| Kite | Index/**futures**/options + stock cash & stock futures | 1m–day | **min/5m ~3+ yr (paginated); day ~20 yr** | Paid + token | Primary. See §3 for caps + rate limits. |
| Kite | Option chain, pre-open | Live | Live only | Same | No historical chain → recorder accumulates forward. |
| NSE | Participant-wise OI (FII/DII/Pro/Client) **index only** | Daily EOD | Multi-yr | Free | Cache hard. |
| NSE | FII/DII cash net (provisional) **index only** | Daily EOD | Multi-yr | Free | **Revised** → store as-of-dated. |
| NSE | F&O bhavcopy: **stock OI + delivery%** per symbol | Daily EOD | Multi-yr | Free | Large; local parquet. |

## 3. Kite historical reality (CORRECTED from v2 — the key fix)

The "60-day" limit is the **maximum span per request**, not total availability. **Paginate** to assemble multi-year series.

| Interval | Max **per request** | Loop strategy |
|----------|--------------------|----------------|
| minute | 60 days | walk back in 60-day windows |
| 3/5/10-minute | 100 days | 100-day windows |
| 15/30-minute | 200 days | 200-day windows |
| 60-minute / hour | 400 days | 400-day windows |
| day / week | 2000 days | one or two requests |

- **Total depth:** Kite serves **~3+ years of minute/5-minute** and **daily back to the 1990s** for many NSE names (varies per instrument; some recently-listed names have less). Print actual first-available dates per instrument in the inventory.
- **Rate limits:** historical API ≈ **3 requests/second**; no documented per-minute/per-day cap, but the API throttles in practice — back off on `Too many requests`. **Fetch once, cache to local parquet, never re-hit a fetched range.** A full universe pull (≈220 stocks × several years × several timeframes + indices + futures) is **tens of thousands of requests** → **hours of wall-clock**. The orchestrator MUST checkpoint and resume.
- **OI in candles:** pass `oi=1` to get open-interest in futures/options candles (used by I4, D3, E1).

## 4. Continuous futures (MANDATORY) — with the intraday catch

Index spot has **no traded volume**; all volume/VWAP/POC/basis logic uses futures. Build a roll-adjusted continuous front-month series per index, **and per F&O stock**, rolling on expiry-minus-N days (N pre-registered, default 1–2). Keep both **back/ratio-adjusted** (for indicators) and **unadjusted per-contract** (for basis math, E1).

**The catch v2 missed:** Kite's `continuous=1` returns **only daily candles** for expired contracts (given a live contract's instrument_token). The instrument master lists tokens only for **live** contracts. Therefore:
- **Daily continuous futures:** use `continuous=1` directly — easy.
- **Intraday continuous futures:** you must **cache the instrument_token of each contract while it is live**, fetch its minute/5-min candles before it expires (or from whatever back-history is available), and **stitch + roll-adjust manually**. Build a `contract_token_registry` that snapshots tokens daily so expired-contract intraday data remains fetchable/cached. Without this, intraday futures strategies (C1/C2/C4, index volume logic) have no clean multi-expiry history.
- **Guard:** a volume/VWAP strategy pointed at index *spot* must raise.

## 5. F&O stock universe (point-in-time)

- Universe = currently F&O-eligible NSE stocks (**~220 — verify live**).
- Build `/data/reference/fno_membership.parquet` with **per-stock add/remove dates**. A backtest includes a stock only for dates it was actually F&O-eligible.
- **Why point-in-time is critical:** stocks are ADDED to F&O *after* becoming large/liquid (they already rallied to qualify) and REMOVED *after* shrinking — today's list on old history bakes in survivorship + look-ahead. Single biggest stock-universe trap.
- **Universe-selection options (Phase 0 recommendation: full universe):** running all ~220 is the cleanest (no cherry-picking, honest false-discovery accounting). If compute-bound, a **stratified subset** (by sector + liquidity tercile, pre-registered) is acceptable; **never** a hand-picked "good stocks" list — that is selection bias before the test even starts.
- Stocks have real cash volume → volume/VWAP on stocks uses **cash-equity OHLCV**; OI uses **stock futures / bhavcopy stock OI**.
- Per-stock FII/DII positioning does **not** exist free → D1/D2 stay INDEX-ONLY; only D3 (OI) and D4 (delivery%) extend to stocks.

## 6. Cross-sectional panel (NEW — enables A7, stock pairs)

Build `/data/panel/` as date × symbol matrices, **eligibility-masked** (NaN where a symbol was not F&O-eligible):
- daily returns, volatility, momentum (multiple lookbacks), liquidity (turnover), sector tag.
- For pair search: aligned price/return panel so cointegration scans run over all eligible same-period pairs.
- **No look-ahead:** ranking/feature columns at date *t* use only data ≤ *t*; eligibility at *t* uses the membership table as-of *t*.

## 7. Corporate actions & survivorship (stocks only)

- Adjust per-stock OHLCV for splits, bonuses, dividends. yfinance adjusted close helps for daily; **Kite intraday needs manual adjustment** against an action calendar — implement and **verify on a known split** (price continuity across the event).
- Handle delisted/merged names via the membership table so backtests don't silently drop losers.
- Index series need none of this — keep the two paths separate.

## 8. Cost & tax module (`costs.py`) — date-aware (VERIFIED current)

`net_cost(segment, side, price, qty, trade_date) -> rupees`. STT schedule (**verified June 2026**; Budget-2026 hike effective **1-Apr-2026**):

| Segment | Side | Until 31-Mar-2026 | From 1-Apr-2026 |
|---------|------|-------------------|-----------------|
| Equity delivery | Buy & Sell | 0.10% | 0.10% (unchanged) |
| Equity intraday | Sell | 0.025% | 0.025% (unchanged) |
| Futures (index/stock) | Sell | 0.02% | **0.05%** |
| Options | Sell premium | 0.10% | **0.15%** |
| Options | Exercise intrinsic | 0.125% | **0.15%** |

Also date-aware: brokerage (Zerodha: ₹0 equity delivery, flat ₹20/order intraday & F&O — verify), NSE exchange transaction charges (segment-specific), GST 18% (on brokerage+exchange+SEBI), SEBI turnover fee, stamp duty (buy side, segment-specific). **Slippage model:**
- Liquid index futures: ≥ half-spread per side.
- Options: scale up with distance-from-ATM and size.
- **Less-liquid F&O stocks:** widen slippage by a liquidity factor (e.g. inverse of median turnover tercile) — mid/small names move on your own order. Make slippage a **function of instrument liquidity and order size**, logged per backtest.

**Lot-size history (point-in-time, VERIFIED):** NIFTY 75 → **65** (eff. Jan-2026, transition from 30-Dec-2025 EOD); BANKNIFTY 35 → **30** (Jan-2026); FINNIFTY 60; Nifty Next 50 = 25; SENSEX 20. Per-stock F&O lot sizes vary and are revised periodically — **load from the NSE `NSE_FO_contract_<ddmmyyyy>.csv.gz` files, point-in-time.**

## 9. Point-in-time expiry calendar (VERIFIED)

- **NIFTY weekly:** Thursday **until 29/31-Aug-2025**, **Tuesday from 1-Sep-2025** (SEBI standardisation; NSE→Tuesday, BSE→Thursday). Monthly = last Tuesday (post-change).
- **BANKNIFTY weekly:** existed **until ~Nov-2024**, then discontinued (SEBI Nov-2024: one weekly per exchange, NSE kept NIFTY). **Monthly only, last Tuesday.**
- **F&O stocks:** monthly only, **last Tuesday** (post-change).
- Holiday → previous trading day. Provide `is_expiry()` / `days_to_expiry()` consulting **dated** rules (do not apply today's Tuesday rule to pre-Sep-2025 history).

## 10. NSE scraper resilience

Realistic browser headers; cookie priming via homepage; session reuse; exponential backoff; local cache so re-runs don't re-hit NSE; on block, log + continue on cache. Isolate NSE file formats behind a small adapter (format changes = one-file fix).

## 11. DEFINITION OF DONE

- [ ] config + skeleton import cleanly.
- [ ] costs.py applies different STT pre/post 1-Apr-2026 for a futures sell and an options sell (printed test); rates verified-live with drift printed.
- [ ] yfinance daily loads for indices AND ≥3 sample stocks; tickers verified.
- [ ] Kite returns index, index-futures, options, stock cash, stock futures — **with pagination assembling >1 year of 5-minute data** for a sample instrument (proves the v2 "60-day" limit is overcome).
- [ ] Daily continuous futures via `continuous=1` for both indices + sample stocks; **intraday continuous futures stitched manually across ≥2 expiries** from the token registry; spot-volume guard raises.
- [ ] fno_membership.parquet returns correct as-of eligibility (a stock not-yet-in-F&O is excluded then); panel is eligibility-masked.
- [ ] Corp-action adjustment verified on a known split (daily AND intraday continuity).
- [ ] NSE scraper retrieves participant OI (index), FII/DII cash (index), bhavcopy with per-stock OI + delivery%; cache hit on re-run demonstrated; fetch orchestrator resumes from checkpoint.
- [ ] Point-in-time store returns as-of (not revised) values.
- [ ] Expiry calendar returns Thursday (pre-Sep-2025) and Tuesday (post) for NIFTY; "no weekly" for post-Nov-2024 BANKNIFTY; last-Tuesday monthly for a sample stock.
- [ ] Lot lookup returns 75 then 65 for NIFTY, 35 then 30 for BANKNIFTY; a sample stock lot resolves point-in-time.
- [ ] Chain recorder writes one snapshot.
- [ ] Data-quality log + printed inventory (index + stock coverage, **realistic per-instrument first-available dates**, gaps, forward-test-only items).

## 12. Hand to Phase 2

Working `/data` (indices + F&O stock universe + cross-sectional panel, corp-action-adjusted, point-in-time) + `costs.py` + `config.yaml`, inventory printed. Phase 2 assumes all of this is leakage-safe.
