# greeks-calculator

Compute first and second-order option Greeks for European options under Black-Scholes-Merton — single contract or aggregated across a multi-leg book.

## What it does

- First-order: Δ (delta), ν (vega), Θ (theta), ρ (rho)
- Second-order: Γ (gamma), vanna, volga, charm, color
- Position-level aggregation across signed legs
- Explicit scaling labels (per vol point, per calendar day, etc.)

## Triggers

- "What's the delta/gamma/theta/vega of …"
- "How much does this lose per day"
- "Gamma scalping" / "vol exposure" / "net Greeks"
- Multi-leg book aggregation requests

## Platform

Works on **Claude Code** and **Claude.ai** (math runs in analysis tool).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-pricing
```

## Reference files

- `references/greeks_reference.md` — Worked examples, position aggregation, intuition guide
