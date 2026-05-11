# yfinance-options

Pull options chains, Greeks, and historicals via the yfinance library.

## Triggers

- "Get the options chain for X"
- "Show me the puts on AAPL"
- "Fetch 0DTE SPY chain"
- "Pull all expiries for QQQ"

## What it does

- Resolves "0DTE / weeklies / monthlies" to actual expiry dates
- Returns a clean chain with mid, spread, OI, volume, IV
- Applies liquidity filters (drops zero-bid, wide-spread, no-volume)
- Surfaces data quality caveats (delayed quotes, stale IV)

## Platform

**Claude Code only** (requires Python + yfinance + network access). Won't run on Claude.ai.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-data-providers
pip install yfinance pandas numpy
```

## Reference files

- `references/api_reference.md` — Full template with batch expiry pulling, IV recomputation
