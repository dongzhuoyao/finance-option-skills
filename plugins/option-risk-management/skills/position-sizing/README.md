# position-sizing

Translate capital + risk tolerance into a number of options contracts. Supports fixed-max-loss and quarter-Kelly methods, plus BP/concentration caps.

## Triggers

- "How many contracts should I trade"
- "What size for this credit spread / iron condor"
- "Kelly fraction for options"
- "Max loss sizing"

## What it does

- Fixed-max-loss: floor(risk_$ / per_contract_max_loss)
- Quarter Kelly: 0.25 · f* · capital / per_contract_max_loss
- Buying-power cap (worst case for short legs)
- Concentration cap (per-underlying %)
- Refuses to size unbounded-loss legs (naked call, ratio short)

## Platform

Works on **Claude Code** and **Claude.ai**.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-risk-management
```
