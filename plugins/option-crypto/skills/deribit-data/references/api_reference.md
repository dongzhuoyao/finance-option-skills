# Deribit Public API — Endpoint Catalog

Base: `https://www.deribit.com/api/v2/public/`
Docs: https://docs.deribit.com/

## Authentication

All endpoints in this skill are public — no API key required. Authenticated endpoints (`/private/...`) are for trading and out of scope for read-only skills.

---

## Endpoints

### `/get_instruments`

Lists all listed instruments for a currency / kind.

| Param | Required | Notes |
|---|---|---|
| `currency` | yes | BTC, ETH, SOL, USDC, EURR, ... |
| `kind` | yes | option, future, future_combo, option_combo, spot |
| `expired` | no | false (default), true — true returns historic instruments |

Returns array of objects with: `instrument_name`, `expiration_timestamp` (ms), `strike` (options), `option_type` (call/put), `contract_size`, `tick_size`, `is_active`.

### `/get_book_summary_by_currency`

Full chain in one call.

| Param | Required |
|---|---|
| `currency` | yes |
| `kind` | yes |

Returns array per instrument with: `instrument_name`, `mark_price`, `mark_iv`, `bid_price`, `ask_price`, `mid_price`, `volume`, `volume_usd`, `open_interest`, `underlying_price`, `interest_rate`, `creation_timestamp`.

Prefer this over per-instrument `ticker` polling for full-chain work.

### `/ticker`

Single instrument quote.

| Param | Required |
|---|---|
| `instrument_name` | yes |

Returns rich snapshot including: `state` (open/closed), `last_price`, `mark_price`, `mark_iv`, `best_bid_price`, `best_ask_price`, `index_price`, `funding_rate` (perp only), `greeks` (options only — delta, gamma, vega, theta, rho), `open_interest`, `volume`.

### `/get_index_price`

| Param | Required | Notes |
|---|---|---|
| `index_name` | yes | btc_usd, eth_usd, btc_usdc, eth_usdc, sol_usd, ... |

Returns `{index_price, estimated_delivery_price}` — index_price is the multi-exchange composite Deribit uses for marks.

### `/get_funding_rate_value`

Average 8h funding for a perpetual over a time range.

| Param | Required |
|---|---|
| `instrument_name` | yes |
| `start_timestamp` | yes (ms) |
| `end_timestamp` | yes (ms) |

For *historical* funding history, use `/get_funding_rate_history` (paginated). For instantaneous funding, `/ticker` on a perpetual returns `funding_rate` (decimal, e.g. 0.0001 = 1bp / 8h ≈ 10.9% / year).

### `/get_volatility_index_data`

DVOL OHLC history.

| Param | Required | Notes |
|---|---|---|
| `currency` | yes | BTC, ETH (DVOL exists for these two) |
| `start_timestamp` | yes (ms) |
| `end_timestamp` | yes (ms) |
| `resolution` | yes | 1, 60, 3600, 43200, 1D — minutes (string) |

Returns `{data: [[timestamp, open, high, low, close], ...]}`. DVOL is a 30-day implied-vol index — Deribit's analogue to VIX.

---

## Websocket Alternative

For real-time, `wss://www.deribit.com/ws/api/v2` supports the same methods plus subscriptions:

```
{"method":"public/subscribe", "params":{"channels":["ticker.BTC-PERPETUAL.100ms"]}}
```

Useful for delta-hedging skills (`option-risk-management`) that need fast spot updates. Out of scope for a read-only data skill — use REST for snapshots.

---

## Pagination

Most public endpoints return full sets without pagination. The exceptions:
- `/get_funding_rate_history` — limit defaults to 744, max ~1000. Page by adjusting `start_timestamp`.
- `/get_last_trades_by_instrument` — `count` parameter, max 1000.

---

## Error Codes (selected)

| Code | Meaning |
|---|---|
| 10004 | Invalid params |
| 10009 | Not enough funds (private only) |
| 10028 | Too many requests (rate limit) |
| 10042 | Settlement in progress (every Friday 08:00 UTC for some markets) |
| 11050 | Bad request |

On 10028, back off exponentially. On settlement-window errors (10042), wait until 08:10 UTC and retry — the option chain reorganizes at expiry.

---

## Coin-Denomination Math

Option prices on Deribit are quoted in the underlying coin:

```
usd_premium  = coin_premium × underlying_price
usd_notional = 1 × underlying_price            # one contract = 1 coin
usd_oi       = open_interest × underlying_price
```

So OI of "10,000 contracts" at BTC=$100,000 is $1B notional, not $10k.

For Greeks, Deribit returns:
- `delta` — w.r.t. the **coin** (not USD). To get USD delta: `delta * (1/S)` per inverse-option math, see `inverse-options-pricing`.
- `vega` — already coin-denominated per 1.00 vol.

If you mix Deribit Greeks with equity-style Greeks without converting, you will be off by ~S× — **always convert explicitly**.
