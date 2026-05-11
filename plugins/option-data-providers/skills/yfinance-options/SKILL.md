---
name: yfinance-options
description: >
  Pull options chains, Greeks, and historicals for any US-listed ticker via the yfinance library.
  Use this skill whenever the user wants options data for a specific ticker — full chain, single
  expiry, ATM strip, or historical option prices. Triggers: "get the options chain", "show AAPL puts",
  "fetch SPY 0DTE chain", "what's the 50-delta strike", "pull all expiries for QQQ", "options for $TICKER".
  Defaults: nearest expiry, both puts and calls, ±20% strikes from spot.
---

# yfinance Options Data

Pull options data via the yfinance Python library. Returns structured chains for downstream skills (`iv-surface`, `vol-skew`, `options-payoff`).

---

## Step 1: Check Tool Availability

```
!`python3 -c "import yfinance, sys; print(f'yfinance {yfinance.__version__} OK')" 2>&1 | head -3`
```

If yfinance is not installed: `pip install yfinance`. Do NOT proceed silently — surface the missing dependency.

---

## Step 2: Resolve User Intent

| Intent | Method | Params |
|---|---|---|
| List all expiries | `Ticker.options` | ticker |
| Single expiry chain | `Ticker.option_chain(expiry)` | ticker, expiry (YYYY-MM-DD) |
| All expiries chain | loop over `options` | ticker |
| Spot price | `Ticker.fast_info['lastPrice']` | ticker |
| Historical underlying | `Ticker.history(...)` | ticker, period |
| Dividends | `Ticker.dividends` | ticker |

For "0DTE chain" → today's date if listed (SPX/SPY/QQQ have daily expiries). If today not listed, return the next trading day.

For "weeklies" → all Fridays in the next 4 weeks.

For "monthlies" → 3rd-Friday expiries only (filter with a calendar check).

---

## Step 3: Code Template

See `references/api_reference.md` for the full template. Skeleton:

```python
import yfinance as yf
import pandas as pd

def get_chain(ticker, expiry=None):
    t = yf.Ticker(ticker)
    spot = t.fast_info['lastPrice']
    if expiry is None:
        expiry = t.options[0]  # nearest
    chain = t.option_chain(expiry)
    calls = chain.calls.copy()
    puts = chain.puts.copy()
    calls['type'] = 'call'
    puts['type'] = 'put'
    df = pd.concat([calls, puts], ignore_index=True)
    df['mid'] = (df['bid'] + df['ask']) / 2
    df['spread'] = df['ask'] - df['bid']
    df['spread_pct'] = df['spread'] / df['mid'].replace(0, float('nan'))
    return {'spot': spot, 'expiry': expiry, 'chain': df}
```

---

## Step 4: Quality Filters (apply BEFORE returning data)

| Filter | Reason |
|---|---|
| Drop bid = 0 or ask = 0 | Stale, no market |
| Drop spread_pct > 0.5 | Illiquid |
| Drop volume = 0 AND OI < 10 | No interest |
| Drop mid < $0.05 | Rounding noise |

If after filtering > 80% of the chain is gone, **report** that the chain is illiquid — don't pretend a clean chain exists.

---

## Step 5: Known Limitations (be explicit)

- **yfinance IV is sometimes stale** (cached) or zero. Re-compute IV downstream via Newton-Raphson if you need accuracy.
- **No Greeks in raw chain**. Use `greeks-calculator` after pricing.
- **Quotes delayed ≥ 15 min** for most users; not for real-time trading.
- **No European-style index detection**: SPX is European, but yfinance does not flag it. Use the `american` flag manually when relevant.
- **Adjusted closes for splits**: historical chain data can have stale strikes after a split. Verify with `Ticker.actions`.

If the user needs real-time, recommend a paid feed (Polygon, IEX, ORATS).

---

## Step 6: Respond / Hand Off

Return the structured chain plus:
1. Spot price and timestamp.
2. Expiry chosen and how many strikes survive filtering.
3. A 5-row preview (ATM ± 2 strikes, both call and put) so the user sees what's there.
4. Suggested next skill: `iv-surface` for fitting, `vol-skew` for skew metrics, `options-payoff` for trade visualization.

---

## Reference Files

- `references/api_reference.md` — Full template with batch expiry pulling, IV recomputation, and rate-limit handling
