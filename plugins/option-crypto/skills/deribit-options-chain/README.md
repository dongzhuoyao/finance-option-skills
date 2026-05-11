# deribit-options-chain

Pull and clean the BTC / ETH / SOL options chain from Deribit. Converts coin-denominated prices to USD, applies liquidity filters, and exposes the ATM strip.

## Triggers

- "Deribit chain"
- "BTC / ETH options chain"
- "Show me 26DEC25 BTC strikes"
- "0DTE BTC options"

## What it does

- One-call full-chain pull via `/get_book_summary_by_currency`
- Parses `BTC-DDMMMYY-K-C/P` instrument names into (coin, expiry, strike, type)
- USD conversion using each contract's `underlying_price`
- Liquidity filters (spread, OI, bid > 0)
- Groups by expiry and surfaces the ATM strip

## Platform

**Claude Code only** (network + Python).

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
pip install requests pandas
```
