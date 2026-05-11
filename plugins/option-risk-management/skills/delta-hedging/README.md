# delta-hedging

Design a delta-hedging schedule for an options position. Decomposes daily P&L into gamma scalp + theta + transaction cost drag.

## Triggers

- "Delta hedge", "gamma scalp"
- "Rebalance band", "how often should I hedge"
- "Realized vol breakeven"
- "Long vol P&L", "short vol P&L"

## What it does

- Computes net Δ, Γ, ν, Θ for the position
- Sizes the rebalance band: α · σ · S · √(dt)
- Expected gamma scalp P&L vs theta
- Realized-vol breakeven
- Transaction cost drag
- Optional Monte Carlo simulation

## Platform

**Claude Code** (Monte Carlo needs numpy). Math without sim runs on Claude.ai.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-risk-management
```
