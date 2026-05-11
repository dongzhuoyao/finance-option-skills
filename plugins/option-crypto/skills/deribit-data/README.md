# deribit-data

Read public market data from Deribit's REST API (no auth required).

## Triggers

- "Deribit data on BTC / ETH / SOL"
- "BTC options chain"
- "Perpetual funding rate"
- "DVOL history"

## What it does

- Wraps `/get_instruments`, `/get_book_summary_by_currency`, `/ticker`, `/get_index_price`, `/get_funding_rate_value`, `/get_volatility_index_data`
- Normalizes instrument-name conventions (`BTC-25APR25-100000-C`)
- Surfaces rate-limit hits explicitly, never silently caches stale data
- Labels coin-denominated prices and converts to USD on demand

## Platform

**Claude Code only** (needs network + Python `requests`).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
pip install requests
```

## Reference files

- `references/api_reference.md` — Endpoint catalog, field schemas, websocket alternatives
