---
name: deribit-options-chain
description: >
  Pull and clean the BTC or ETH options chain from Deribit, with proper inverse-quote handling
  and downstream-ready columns. Use this skill whenever the user wants the full Deribit options chain,
  a single expiry, an ATM strip, or a chain filtered by liquidity. Triggers: "Deribit chain",
  "BTC options chain", "ETH chain", "show me 26DEC25 BTC strikes", "what's on Deribit for BTC right now",
  "0DTE BTC options". Defaults: currency = BTC, nearest 4 weekly + nearest 4 monthly expiries,
  bid/ask spread filter 30%.
---

# Deribit Options Chain (BTC / ETH / SOL)

Pull the full options chain from Deribit, normalize fields, and convert coin-denominated prices to USD for downstream skills.

---

## Step 1: Pull via `deribit-data`

Use `deribit-data` to call `/get_book_summary_by_currency?currency=BTC&kind=option` — one request returns all live option instruments.

```python
import requests, pandas as pd
r = requests.get("https://www.deribit.com/api/v2/public/get_book_summary_by_currency",
                 params={"currency": "BTC", "kind": "option"}, timeout=15)
r.raise_for_status()
chain = pd.DataFrame(r.json()["result"])
```

If the request fails, **stop** and surface the error. Do not fall back to cached data without telling the user the data is stale.

---

## Step 2: Parse Instrument Name

Each `instrument_name` decodes to (coin, expiry, strike, type):

```python
def parse_name(name):
    parts = name.split('-')  # e.g. ['BTC', '26DEC25', '120000', 'C']
    coin, exp_str, strike_str, opt_type = parts
    expiry = pd.to_datetime(exp_str, format='%d%b%y')
    return coin, expiry, int(strike_str), opt_type
```

Add columns `expiry`, `strike`, `option_type`, `DTE` (calendar days), `T` (years).

---

## Step 3: Convert to USD

Deribit quotes premiums in the underlying. For downstream BS comparison or USD P&L:

```python
chain['underlying'] = chain['underlying_price']     # spot at the quote time
chain['usd_mark']   = chain['mark_price']  * chain['underlying']
chain['usd_bid']    = chain['bid_price']   * chain['underlying']
chain['usd_ask']    = chain['ask_price']   * chain['underlying']
chain['usd_mid']    = (chain['usd_bid'] + chain['usd_ask']) / 2
chain['spread_pct'] = (chain['ask_price'] - chain['bid_price']) / chain['mark_price'].replace(0, float('nan'))
chain['moneyness']  = chain['strike'] / chain['underlying']
```

**Critical**: `mark_iv` from Deribit is already a decimal (0.65 = 65%). Do not re-divide by 100.

---

## Step 4: Quality Filters

| Filter | Why |
|---|---|
| Drop `state == 'closed'` | After-expiry instruments still show up |
| Drop `bid_price == 0` or `ask_price == 0` | No two-sided market |
| Drop `spread_pct > 0.30` | Illiquid, mid is noise |
| Drop `usd_mid < 5` | Rounding noise dominates |
| Drop `open_interest < 5` AND `volume == 0` | No interest |

For 0DTE BTC chains, the filters are still strict because near-expiry deep OTM options trade for $0.00 to $0.05 BTC bids only. Don't relax filters silently — instead report "after filtering: N contracts survive out of M".

---

## Step 5: Group by Expiry

Standard Deribit expiry cadence:
- Daily: most days (since 2023)
- Weekly: every Friday 08:00 UTC
- Monthly: last Friday of the month
- Quarterly: last Friday of Mar/Jun/Sep/Dec
- 0DTE: today, if listed

Top-of-mind expiries the user usually wants:
1. Next Friday (weekly)
2. Last Friday of current month (monthly)
3. Next quarterly (often most liquid)

If the user just says "the BTC chain", show 1, 2, 3 — and tell them what you picked.

---

## Step 6: ATM Strip

For each expiry, identify the 5 strikes closest to spot (call + put). These are the most liquid and used for ATM IV, term-structure work, and most retail strategies.

```python
def atm_strip(df, expiry, spot, n_strikes=5):
    ex = df[df['expiry'] == expiry].copy()
    ex['dist'] = (ex['strike'] - spot).abs()
    strikes = ex.sort_values('dist')['strike'].unique()[:n_strikes]
    return ex[ex['strike'].isin(strikes)].sort_values(['strike','option_type'])
```

---

## Step 7: Greeks Pass-Through

Unlike yfinance, Deribit's `/ticker` returns Greeks for each instrument. For full chain Greeks, you'd need one `/ticker` call per instrument (rate-limited). Alternatives:
1. Recompute via `inverse-options-pricing` skill (faster, requires IV and spot).
2. Only fetch Greeks for the ATM strip via `/ticker`.

**Caveat**: Deribit Greeks are in **coin units**, not USD units. See `inverse-options-pricing` for conversion math. Do not mix them with equity-style Greeks without converting.

---

## Step 8: Respond / Hand Off

Return the structured chain plus:
1. Spot and as-of timestamp.
2. Expiries picked + filter survival counts.
3. ATM strip preview.

Hand off to:
- `inverse-options-pricing` for theoretical pricing
- `iv-surface` (with crypto caveat — see that skill's notes) for fitting
- `gex-max-pain` for positioning
- `options-payoff` for trade visualization (use `usd_mid` as premium for inverse-quote conversion)
