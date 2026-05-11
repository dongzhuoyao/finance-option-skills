# dvol-index

Read Deribit's DVOL index (BTC + ETH) — the crypto analogue of CBOE's VIX — for vol regime context and IV-RV spread analysis.

## Triggers

- "DVOL", "crypto VIX"
- "BTC / ETH vol regime"
- "Is crypto vol cheap"
- "IV vs realized vol for BTC"

## What it does

- Current DVOL level + 1y percentile
- Regime classification (calm / normal / elevated / stressed / crisis)
- IV-RV spread (vol carry context)
- Term-structure read (via ATM IVs from `deribit-options-chain`)

## Platform

**Claude Code only** (network + Python).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
pip install requests pandas numpy
```
