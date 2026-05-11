# options-chain-reader

Parse broker option-chain screenshots (IBKR / TastyTrade / ToS / Robinhood / Webull) into a structured legs table.

## Triggers

- Uploading a broker screenshot
- "Parse this position"
- "What strategy is this"
- "Analyze my IBKR / TastyTrade / ToS trade"

## What it does

- Identifies the broker by layout/branding
- Extracts per-leg fields (type, strike, expiry, side, qty, fill, IV, Greeks)
- Classifies the strategy (single leg / vertical / iron condor / butterfly / …)
- Sanity-checks against the broker's stated net debit/credit
- Outputs structured JSON for downstream skills

## Platform

Works on **Claude.ai** and **Claude Code** (vision-capable).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-data-providers
```
