---
name: options-chain-reader
description: >
  Parse a broker option chain screenshot or pasted text into a structured legs table.
  Use this skill when the user uploads a screenshot from IBKR / TastyTrade / ToS / Robinhood / Schwab
  or pastes a chain dump and asks to "parse this", "analyze this position", "what trade is this",
  or wants Greeks / payoff from a broker image. Triggers: any chain screenshot, "IBKR position",
  "TastyTrade trade ticker", "ToS analyzer page", "what does my position say".
  Do NOT use this to fetch fresh data — use `yfinance-options` for that.
---

# Broker Option Chain Reader

Convert a broker screenshot into a structured legs table the downstream skills (`greeks-calculator`, `options-payoff`) can consume.

---

## Step 1: Identify the Broker

Look for layout / branding:

| Broker | Tell |
|---|---|
| **IBKR (TWS)** | Beige/grey theme, "Bid", "Ask", "Last", "Vol", "OI", "IV", "Delta" columns, expiry as `YYYYMMDD` |
| **TastyTrade** | Dark theme, large bid/ask boxes, strike in the middle, calls left / puts right |
| **ToS (Schwab)** | Dark theme, "ProbITM" column, "Theo Price" column, expiry as e.g. `JAN 17 25` |
| **Robinhood** | Mobile-first, single-side at a time, fewer columns |
| **Webull** | Light-or-dark, multi-leg structure builder |

If you can't identify the broker confidently, **ask the user** — don't guess. Misreading column order silently produces wrong P&L.

---

## Step 2: Extract Legs

For each leg, capture:

| Field | Required | Notes |
|---|---|---|
| Underlying ticker | yes | Top of screenshot |
| Type (call/put) | yes | Column header or left/right grouping |
| Strike | yes | Listed K |
| Expiry | yes | Normalize to `YYYY-MM-DD` |
| Side (buy/sell) | yes | + / − or BUY/SELL or filled-trade direction |
| Quantity | yes | Number of contracts |
| Fill price | yes | The price you paid (or received, for shorts) |
| Multiplier | usually 100 | 100 for US equity options |
| Current bid/ask | optional | If shown, capture for re-pricing |
| IV | optional | Capture if shown |
| Greeks (Δ, Γ, Θ, ν) | optional | Capture if shown |

---

## Step 3: Identify the Strategy

Match the leg structure to a known strategy:

| Legs | Strategy |
|---|---|
| 1 long call | Long call |
| 1 short call | Naked short call |
| 1 long put | Long put |
| 1 short put | Cash-secured / naked put |
| 1 long, 1 short, same type, same expiry, different strikes | Vertical spread |
| 1 long, 1 short, same type, same strike, different expiries | Calendar |
| 1 long, 1 short, same type, different strike, different expiry | Diagonal |
| 1 long call + 1 long put, same K, same expiry | Long straddle |
| 1 long OTM call + 1 long OTM put | Long strangle |
| Buy K1 + sell 2×K2 + buy K3, same expiry | Butterfly |
| Sell put spread + sell call spread, 4 strikes | Iron condor |
| Stock + short call | Covered call |
| Stock + long put | Protective put |
| Asymmetric legs | Custom — list as-is |

---

## Step 4: Sanity Checks

1. **Sum of premiums** should match the net debit/credit shown on the screenshot (if visible). If off by > 1% per leg, you read at least one fill price wrong. **Stop and re-read** — don't auto-correct silently.
2. **Multiplier**: equity options are 100. If the screenshot says "MULT 100" or "x100", confirm. SPX is 100, but futures options vary.
3. **Days to expiry**: compute from today; if the broker shows a different DTE (rare), use the broker's — it accounts for early closes.

---

## Step 5: Structured Output

Return:

```json
{
  "underlying": "AAPL",
  "strategy": "iron_condor",
  "as_of": "2026-05-11T15:34:00Z",
  "legs": [
    {"type": "put",  "strike": 170, "expiry": "2026-06-19", "side": "buy",  "qty": 1, "fill": 0.45, "iv": 0.27, "delta": -0.08},
    {"type": "put",  "strike": 175, "expiry": "2026-06-19", "side": "sell", "qty": 1, "fill": 0.85, "iv": 0.26, "delta": -0.15},
    {"type": "call", "strike": 200, "expiry": "2026-06-19", "side": "sell", "qty": 1, "fill": 0.90, "iv": 0.24, "delta":  0.16},
    {"type": "call", "strike": 205, "expiry": "2026-06-19", "side": "buy",  "qty": 1, "fill": 0.50, "iv": 0.25, "delta":  0.09}
  ],
  "net_debit_credit": +0.80,
  "multiplier": 100,
  "max_profit_observed": 80,
  "max_loss_observed": 420
}
```

---

## Step 6: Hand Off

After parsing:
- Recommend `options-payoff` to visualize the P&L curve.
- Recommend `greeks-calculator` to compute net Greeks.
- If IV looks stale or inconsistent across legs, recommend `iv-surface` for a fresh fit.

---

## What This Skill Won't Do

- Decide whether the trade is good. That's `strategy-selector` or human judgment.
- Submit orders. Read-only.
- Fetch real-time data. Use `yfinance-options` for that.
