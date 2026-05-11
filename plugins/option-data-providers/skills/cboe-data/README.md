# cboe-data

Read CBOE volatility indices (VIX, VVIX, SKEW, VIX9D/3M/6M, put/call) and interpret market vol regime.

## Triggers

- "VIX / VVIX / SKEW level"
- "Vol regime", "Is the market complacent / stressed"
- "VIX term structure", "frontend stress"
- "Put/call ratio"

## What it does

- Snapshot of VIX-family indices via yfinance
- Term-structure ratios (VIX9D / VIX, VIX / VIX3M, VIX / VIX6M)
- 1y percentile context
- Cross-index regime classification

## Platform

**Claude Code only** (requires yfinance + network).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-data-providers
pip install yfinance
```
