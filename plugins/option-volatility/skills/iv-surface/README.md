# iv-surface

Fit and visualize the implied volatility surface for any ticker. Detects arbitrage violations and surfaces cheap/rich contracts.

## What it does

- Pulls the full options chain (via `yfinance-options`)
- Solves for implied vol on each contract (Newton-Raphson on Black-Scholes)
- Fits SVI (index) or SABR (equity) per expiry
- Runs calendar/butterfly/wing arbitrage checks
- Surface heatmap + per-expiry smile chart
- Lists top dislocations (cheapest/richest residuals)

## Triggers

- "Show me the IV surface for X"
- "Is the vol surface arb-free"
- "Find dislocated options"
- "Fit SVI / SABR"
- "Scan the chain for cheap vol"

## Platform

Works on **Claude Code** (network access for yfinance). Claude.ai cannot run yfinance directly but can render the result.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-volatility
pip install yfinance numpy scipy
```

## Reference files

- `references/iv_reference.md` — SVI / SABR derivations, Newton-Raphson, arbitrage tests
