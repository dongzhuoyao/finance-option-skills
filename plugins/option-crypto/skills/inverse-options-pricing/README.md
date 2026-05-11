# inverse-options-pricing

Price coin-settled (inverse) options the way Deribit quotes them — premium and payoff in BTC/ETH, not USD. Converts Greeks between coin-units and USD-units.

## Triggers

- "Inverse option", "coin-settled"
- "Deribit pricing"
- "Why is Deribit delta different from BS delta"
- "Convert Deribit Greeks to USD"

## What it does

- Vanilla USD BS price, then divides by S for coin-quoted price (matches Deribit mark)
- Inverse Greeks: delta_coin, vega_coin, theta_coin, gamma_coin formulas
- USD-equivalent Greeks for hedge sizing
- Put-call parity check in inverse form
- Bound checks (C_coin ≤ 1)

## Platform

Works on **Claude Code** and **Claude.ai** (math runs in analysis tool).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
```

## Reference files

- `references/inverse_math.md` — Full derivation, USD vs coin numeraire, ETH staking yield correction
