# binomial-pricing

Price American options via Cox-Ross-Rubinstein binomial tree. Reports the early-exercise premium and the optimal-exercise boundary.

## What it does

- CRR binomial tree pricing for American calls and puts
- European cross-check for convergence (Black-Scholes residual)
- Early-exercise premium = American − European
- Optimal-exercise boundary at t = 0

## Triggers

- "American option", "early exercise", "dividend exercise"
- "Should I exercise this option early"
- "CRR tree", "binomial", "lattice"
- Dividend-paying American calls

## Platform

Works on **Claude Code** and **Claude.ai** (math runs in analysis tool).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-pricing
```
