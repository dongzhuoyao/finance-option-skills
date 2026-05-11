# gex-max-pain

Compute dealer Gamma Exposure (GEX) and Max Pain strike for BTC/ETH at any Deribit expiry.

## Triggers

- "GEX", "gamma exposure"
- "Max pain"
- "BTC expiry magnet"
- "Where will BTC pin on Friday"
- "Dealer gamma"

## What it does

- Max-pain strike: minimizes total option-holder payoff at expiry
- GEX: sum of dealer gamma exposure across the chain (sign assumed; flagged as such)
- Gamma walls by strike (horizontal bar chart)
- Interpretation tied to vol regime

## Caveats (built into the skill output)

- Dealer positioning is unknowable without flow data — signs are heuristic
- Max pain is a folk indicator, not a forecast
- Refuses to run on thin chains (< 200 total OI)

## Platform

**Claude Code only** (network + Python + chain math).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
```
