# options-payoff

Generate interactive options payoff curve charts with dynamic parameter controls.

## What it does

This skill renders a fully interactive HTML widget showing:

- **Expiry payoff curve** (dashed gray line) — intrinsic value at expiration
- **Theoretical value curve** (solid colored line) — Black-Scholes price at current DTE/IV
- Dynamic sliders for all key parameters (strikes, premium, IV, DTE, spot price)
- Real-time stats: max profit, max loss, breakevens, current P&L at spot

## Supported strategies

| Strategy | Legs |
|---|---|
| Butterfly | Buy K1, Sell 2×K2, Buy K3 |
| Broken-wing butterfly | Buy K1, Sell 2×K2, Buy K3 (asymmetric) |
| Vertical spread | Buy K1, Sell K2 (same expiry) |
| Calendar spread | Buy far-expiry K, Sell near-expiry K |
| Diagonal spread | Buy far K1, Sell near K2 |
| Iron condor | Sell K2/K3, Buy K1/K4 wings |
| Jade lizard | Sell put + sell call spread |
| Straddle | Buy Call K + Buy Put K |
| Strangle | Buy OTM Call + Buy OTM Put |
| Covered call | Long 100 shares + Sell Call K |
| Protective put | Long 100 shares + Buy Put K |
| Naked put | Sell Put K |
| Ratio spread | Buy 1×K1, Sell N×K2 |

For unlisted strategies, the skill uses `custom` mode — decomposing into individual legs and summing their P&Ls.

## Triggers

- Describing an options strategy ("show me a bull call spread")
- Uploading a screenshot from a broker (IBKR, TastyTrade, Robinhood, ToS)
- Mentioning strike prices, premiums, or expiry dates
- Asking to "show me the payoff", "draw the P&L curve", or "what does this trade look like"

## Platform

Works on **Claude.ai** (via the built-in `show_widget` tool) or on **Claude Code**.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-strategies
npx skills add dongzhuoyao/finance-option-skills --skill options-payoff
```

## Reference files

- `references/strategies.md` — Detailed payoff formulas and edge cases for each strategy type
- `references/bs_code.md` — Copy-paste ready Black-Scholes JS implementation with normCDF

## Credit

Adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills) (MIT) `options-payoff` skill, extended with diagonal spreads, jade lizards, broken-wing butterflies, and protective puts.
