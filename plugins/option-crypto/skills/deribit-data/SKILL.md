---
name: deribit-data
description: >
  Read public market data from Deribit's REST API — instruments, ticker, order book, index price,
  funding rate, historical DVOL — no auth required. Use this skill whenever the user asks for
  Deribit data on BTC, ETH, SOL, or any other Deribit currency, including option instruments,
  perpetual funding, dated futures basis, or the volatility index. Triggers: "Deribit",
  "BTC options", "ETH options", "BTC funding rate", "perpetual basis", "DVOL", "Deribit ticker",
  "options chain on Deribit", "BTC-PERPETUAL", "BTC-25DEC25-100000-C".
  Activate even with partial input — default currency = BTC, default kind = option.
---

# Deribit Public Data Reader

Pull market data from Deribit's free, no-auth public REST API. Wrapper around the most useful endpoints, with normalization for downstream skills.

Base URL: `https://www.deribit.com/api/v2/public/`

---

## Step 1: Resolve Intent

| User wants | Endpoint | Params |
|---|---|---|
| List option instruments | `/get_instruments` | currency=BTC, kind=option, expired=false |
| List perp / futures instruments | `/get_instruments` | currency=BTC, kind=future |
| Full chain summary (all strikes/expiries) | `/get_book_summary_by_currency` | currency=BTC, kind=option |
| Single instrument quote | `/ticker` | instrument_name=BTC-25APR25-100000-C |
| Spot index | `/get_index_price` | index_name=btc_usd |
| Funding rate | `/get_funding_rate_value` | instrument_name=BTC-PERPETUAL, start_timestamp, end_timestamp |
| DVOL history | `/get_volatility_index_data` | currency=BTC, start_timestamp, end_timestamp, resolution=1 |

If the user names an instrument that isn't on Deribit (e.g. SOL options before 2024 listing), surface that — don't guess.

---

## Step 2: Connectivity Check

```
!`curl -sS --max-time 3 https://www.deribit.com/api/v2/public/get_index_price?index_name=btc_usd 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); print(f'BTC = \${d[\"result\"][\"index_price\"]:,.0f}')" || echo "DERIBIT_UNAVAILABLE"`
```

If the API is unreachable, **stop** and surface the failure — do not substitute cached data without telling the user.

---

## Step 3: Request Template (Python)

```python
import requests
BASE = "https://www.deribit.com/api/v2/public"

def deribit_get(path, **params):
    r = requests.get(f"{BASE}/{path}", params=params, timeout=10)
    r.raise_for_status()
    j = r.json()
    if 'error' in j:
        raise RuntimeError(f"Deribit error {j['error']}")
    return j['result']

# Examples:
instruments = deribit_get("get_instruments", currency="BTC", kind="option", expired="false")
chain       = deribit_get("get_book_summary_by_currency", currency="BTC", kind="option")
spot        = deribit_get("get_index_price", index_name="btc_usd")['index_price']
ticker      = deribit_get("ticker", instrument_name="BTC-25APR25-100000-C")
```

For DVOL or funding history (range queries), supply `start_timestamp` and `end_timestamp` in **milliseconds** since epoch.

---

## Step 4: Instrument-Name Convention (memorize)

```
{COIN}-{DDMMMYY}-{STRIKE}-{C|P}        e.g.  BTC-26DEC25-120000-C
{COIN}-PERPETUAL                       perpetual swap
{COIN}-{DDMMMYY}                       dated future
```

Months are 3-letter UPPERCASE English (JAN/FEB/.../DEC). Strike is integer USD. Type is `C` (call) or `P` (put).

If a user gives "BTC 120k Dec calls" → resolve to `BTC-26DEC25-120000-C` (the last Friday closest to year-end). Always confirm if ambiguous (multiple Dec expiries listed).

---

## Step 5: Rate Limits (be polite)

Public endpoints rate limit: ~20 req/sec per IP. For full-chain pulls (300+ instruments per currency), batch via `get_book_summary_by_currency` (one call returns the whole chain) rather than per-instrument `ticker` calls.

If you hit 429 → back off exponentially (2s, 4s, 8s). After 3 failed retries, **stop and report** — don't pretend the chain is empty.

---

## Step 6: Field Notes

For options (returned by `get_book_summary_by_currency`):

| Field | Meaning |
|---|---|
| `instrument_name` | The contract name |
| `mark_price` | Deribit's mark in **BTC/ETH** (inverse-quoted) |
| `mark_iv` | Mark implied vol, decimal (e.g. 0.65 = 65%) |
| `bid_price` / `ask_price` | Top of book in coin units |
| `volume` / `volume_usd` | 24h volume |
| `open_interest` | OI in contracts (1 contract = 1 BTC notional for BTC, 1 ETH for ETH) |
| `underlying_price` | Spot reference at the quote time |

**Coin quoting**: `mark_price = 0.04` means premium ≈ 0.04 BTC. To get USD: `usd_premium = mark_price * underlying_price`.

---

## Step 7: Respond / Hand Off

Return structured JSON for the requested instrument(s). Always include:
1. `as_of` timestamp (UTC).
2. Underlying spot.
3. Field-by-field with units explicitly labeled.

Hand off:
- For chain analysis → `deribit-options-chain`
- For BS pricing under inverse settlement → `inverse-options-pricing`
- For DVOL regime → `dvol-index`
- For GEX / max-pain → `gex-max-pain`

---

## Reference Files

- `references/api_reference.md` — Full endpoint catalog with field schemas, pagination, websocket alternatives
