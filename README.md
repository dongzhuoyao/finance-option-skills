# Finance Option Skills

> [!WARNING]
> This project is for educational and informational purposes only. Nothing here constitutes financial advice. Options trading involves substantial risk of loss and is not suitable for all investors. Always do your own research and consult a qualified financial advisor before making investment decisions.

A collection of agent skills focused on **options trading and derivatives**, following the [Agent Skills](https://agentskills.io) open standard. Inspired by and structured after [himself65/finance-skills](https://github.com/himself65/finance-skills), but specialized for options: pricing, Greeks, multi-leg strategies, implied volatility, and risk management.

## Quick Start

### Claude Code — All Plugins

```bash
npx plugins add dongzhuoyao/finance-option-skills
```

### Claude Code — Individual Plugins

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-pricing
npx plugins add dongzhuoyao/finance-option-skills --plugin option-strategies
npx plugins add dongzhuoyao/finance-option-skills --plugin option-volatility
npx plugins add dongzhuoyao/finance-option-skills --plugin option-data-providers
npx plugins add dongzhuoyao/finance-option-skills --plugin option-risk-management
npx plugins add dongzhuoyao/finance-option-skills --plugin option-skill-creator
```

### Claude Code — Individual Skills

```bash
npx skills add dongzhuoyao/finance-option-skills
```

### Other Agents

```bash
npx skills add dongzhuoyao/finance-option-skills -a <agent-name>
```

## Available Skills

### Option Pricing (`option-pricing`)

Closed-form and numerical pricing for European and American options, plus the full Greek family.

| Skill | Description |
|---|---|
| [black-scholes](plugins/option-pricing/skills/black-scholes/) | Black-Scholes-Merton pricing for European calls/puts — with dividend yield, put-call parity sanity checks |
| [greeks-calculator](plugins/option-pricing/skills/greeks-calculator/) | First and second-order Greeks — delta, gamma, theta, vega, rho, vanna, volga, charm |
| [binomial-pricing](plugins/option-pricing/skills/binomial-pricing/) | Cox-Ross-Rubinstein binomial tree for American options and early-exercise premium estimation |

### Option Strategies (`option-strategies`)

Multi-leg payoff analysis and strategy selection by market view.

| Skill | Description |
|---|---|
| [options-payoff](plugins/option-strategies/skills/options-payoff/) | Interactive payoff curve widget — verticals, butterflies, condors, calendars, straddles, ratios |
| [strategy-selector](plugins/option-strategies/skills/strategy-selector/) | Match user's directional/vol view + risk tolerance to the right multi-leg structure |

### Option Volatility (`option-volatility`)

Implied volatility analysis across strike and time dimensions.

| Skill | Description |
|---|---|
| [iv-surface](plugins/option-volatility/skills/iv-surface/) | Fit and visualize the full IV surface — SVI/SABR parameterizations, no-arbitrage checks |
| [vol-skew](plugins/option-volatility/skills/vol-skew/) | Strike-axis skew — 25-delta risk reversal, butterfly, put/call skew interpretation |
| [vol-term-structure](plugins/option-volatility/skills/vol-term-structure/) | Time-axis term structure — contango vs backwardation, earnings vol crush forecasting |

### Option Data Providers (`option-data-providers`)

External data sources for options chains and volatility indices.

| Skill | Description |
|---|---|
| [yfinance-options](plugins/option-data-providers/skills/yfinance-options/) | Pull options chains, Greeks, and historicals via yfinance — full snapshot or single expiry |
| [options-chain-reader](plugins/option-data-providers/skills/options-chain-reader/) | Parse broker option chain screenshots (IBKR / TastyTrade / ToS / Robinhood) into structured legs |
| [cboe-data](plugins/option-data-providers/skills/cboe-data/) | Read CBOE volatility indices — VIX, VVIX, SKEW, put/call ratio, VIX9D/VIX3M for term structure |

### Option Risk Management (`option-risk-management`)

Hedging, sizing, and portfolio-level Greeks for options books.

| Skill | Description |
|---|---|
| [delta-hedging](plugins/option-risk-management/skills/delta-hedging/) | Build a delta-hedging schedule — rebalance bands, gamma scalping P&L attribution |
| [position-sizing](plugins/option-risk-management/skills/position-sizing/) | Kelly-fraction and max-loss-based sizing for premium sellers (cash-secured puts, credit spreads) |
| [portfolio-greeks](plugins/option-risk-management/skills/portfolio-greeks/) | Aggregate Greeks across a multi-position book and flag concentration risks |

### Option Skill Creator (`option-skill-creator`)

| Skill | Description |
|---|---|
| [skill-creator](plugins/option-skill-creator/skills/skill-creator/) | Create, evaluate, and iterate on new options-focused skills with a quality rubric |

## Credit

Repository layout, plugin marketplace design, and many SKILL.md conventions are adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills) (MIT). This repo narrows that template to focus on options/derivatives.

## License

MIT
