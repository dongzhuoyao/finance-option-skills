# portfolio-greeks

Aggregate Greeks across a multi-position options book and flag concentration risks.

## Triggers

- "Portfolio Greeks", "book Greeks"
- "Net delta / vega / gamma / theta"
- "Concentration risk", "stress test my book"
- "What if X moves 5%"

## What it does

- Per-position Greeks via `greeks-calculator`
- Per-underlying and book-level aggregation (dollar Greeks)
- Concentration flags (BP, sector, expiry, vol/gamma combo)
- 5 canonical stress scenarios with linear P&L approx
- One-line risk summary

## Platform

Works on **Claude Code** and **Claude.ai**.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-risk-management
```
