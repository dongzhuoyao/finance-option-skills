# black-scholes

Price European call and put options using the Black-Scholes-Merton closed-form model with continuous dividend yield support.

## What it does

Given (S, K, T, r, q, σ) the skill returns:

- The theoretical option price
- Intrinsic vs extrinsic decomposition
- Put-call parity verification
- A side-by-side comparison vs the market quote (if provided)

## Triggers

- "What's the fair value of …"
- "Price this call/put"
- "Is this option overpriced/underpriced?"
- Mentions of Black-Scholes, BSM, theoretical price, put-call parity, intrinsic/extrinsic

## Platform

Works on **Claude Code** (uses Python/scipy if available, falls back to inline normCDF) and **Claude.ai** (math can run in Python/JS analysis tool).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-pricing
# or just this skill
npx skills add dongzhuoyao/finance-option-skills --skill black-scholes
```

## Reference files

- `references/bs_formulas.md` — Full derivation, scipy-free normCDF, dividend handling, batch pricing template
