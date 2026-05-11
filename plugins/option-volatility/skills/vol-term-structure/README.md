# vol-term-structure

ATM IV across expiries — detect contango / backwardation, flag earnings vol bumps, surface curve trades.

## Triggers

- "Term structure", "vol curve", "contango", "backwardation"
- "Earnings vol crush"
- "Near vol vs far vol"
- "VIX9D vs VIX vs VIX3M"

## What it does

- ATM IV for all listed expiries (up to 1Y)
- Classifies shape (contango / backwardation / humped)
- Flags event-driven vol jumps
- Suggests curve trades when appropriate

## Platform

Works on **Claude Code** (yfinance required).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-volatility
```
