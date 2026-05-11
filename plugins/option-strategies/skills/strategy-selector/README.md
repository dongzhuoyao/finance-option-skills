# strategy-selector

Recommend the best-fit options strategy given a user's directional view, volatility view, risk tolerance, and capital.

## What it does

- Maps a (direction × vol view) thesis to a ranked list of candidate structures
- Applies filters for risk-tolerance, capital size, and experience level
- Hands off to `options-payoff` to visualize the top pick

## Triggers

- "What strategy should I use to play X"
- "How should I express a [bullish/bearish/neutral] view on Y"
- "I want to bet on a move but cap my loss"
- "I want to collect premium on Z"

## Platform

Works on **Claude Code** and **Claude.ai**.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-strategies
```
