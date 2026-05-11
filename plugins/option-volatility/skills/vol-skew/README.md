# vol-skew

Strike-axis vol skew analysis: 25-delta risk reversal, butterfly, put-call slope, with 1-year historical percentile context.

## Triggers

- "Vol skew", "risk reversal", "25d RR", "25d butterfly"
- "Put skew rich/cheap"
- "Tail risk premium"
- "Skew percentile vs history"

## What it does

- Picks the expiry closest to user's DTE
- Interpolates to 25-delta strikes on both sides
- Computes RR, butterfly, slope, ATM IV
- 1y rolling percentile context
- Flags actionable extremes

## Platform

Works on **Claude Code** (yfinance required). Output charts render on Claude.ai too.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-volatility
```
