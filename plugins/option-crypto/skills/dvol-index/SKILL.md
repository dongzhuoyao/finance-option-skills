---
name: dvol-index
description: >
  Read Deribit's DVOL index (BTC and ETH) — the crypto analogue of CBOE's VIX — for vol regime
  context, term-structure flags, and historical percentile comparisons. Use this skill whenever
  the user asks about crypto vol regime, BTC vs ETH vol, DVOL level vs history, or wants
  a vol-context read before crypto-options trades. Triggers: "DVOL", "BTC DVOL", "ETH DVOL",
  "crypto VIX", "BTC vol regime", "is crypto vol cheap", "implied vol vs realized for BTC".
  Defaults: BTC currency, trailing 1y window for percentile.
---

# DVOL — Deribit Volatility Index Reader

DVOL is Deribit's 30-day forward implied volatility index, computed from listed BTC/ETH option prices using a variance-swap replication. Functionally it's "the VIX of crypto".

---

## Step 1: Pull Current Level

```
!`curl -sS --max-time 3 "https://www.deribit.com/api/v2/public/get_volatility_index_data?currency=BTC&start_timestamp=$(($(date +%s) * 1000 - 86400000))&end_timestamp=$(($(date +%s) * 1000))&resolution=43200" 2>/dev/null | python3 -c "import json,sys; d=json.load(sys.stdin); rows=d['result']['data']; print(f'BTC DVOL last: {rows[-1][4]:.1f}' if rows else 'DVOL: no data')" || echo "DVOL_UNAVAILABLE"`
```

For ETH, change `currency=BTC` to `currency=ETH`.

---

## Step 2: Pull History

```python
import requests, pandas as pd
from datetime import datetime, timedelta

def get_dvol(currency='BTC', days=365, resolution='43200'):
    end = int(datetime.utcnow().timestamp() * 1000)
    start = end - days * 86400 * 1000
    r = requests.get(
        "https://www.deribit.com/api/v2/public/get_volatility_index_data",
        params={
            "currency": currency,
            "start_timestamp": start,
            "end_timestamp": end,
            "resolution": resolution,
        }, timeout=15)
    r.raise_for_status()
    rows = r.json()['result']['data']
    df = pd.DataFrame(rows, columns=['ts', 'open', 'high', 'low', 'close'])
    df['ts'] = pd.to_datetime(df['ts'], unit='ms')
    return df
```

Resolution values: `1` (1min), `60` (1h), `3600` (1h... wait, see below), `43200` (12h), `D` (daily). Deribit's resolution is in **seconds** as a string. `43200` = 12h. Use `D` for daily bars over a 1y window.

If the API fails, **stop** — don't substitute cached or approximate values.

---

## Step 3: Classify Level

DVOL ranges (BTC, historical 2021-now):

| Regime | DVOL |
|---|---|
| Calm (rare) | < 35 |
| Normal | 35 – 55 |
| Elevated | 55 – 75 |
| Stressed | 75 – 100 |
| Crisis | > 100 (e.g., LUNA, FTX, March 2020) |

ETH DVOL tends to run 10-20 vol points higher than BTC due to ETH's higher realized vol and lower options liquidity.

Always show 1y percentile alongside the absolute level — "DVOL = 52" means very different things at the 10th vs 90th percentile of the year.

---

## Step 4: Term Structure (DVOL alone is 30-day — for a curve, use ATM IVs from `deribit-options-chain`)

DVOL is a single 30-day point. To get the full vol curve, use `deribit-options-chain` to compute ATM IV per expiry. Compare the 7d ATM IV vs DVOL vs 90d ATM IV for contango/backwardation read.

```
short_long_ratio = atm_iv_7d / atm_iv_90d
```

| Pattern | Read |
|---|---|
| Short < Long (contango) | Calm regime, vol-sellers' market |
| Short > Long (backwardation) | Frontend stress, vol-buyers' market |
| Curve kink at expiry near a known event | Event vol — token unlock, halving, scheduled upgrade |

Halvings (BTC ~every 4 years) and ETH protocol upgrades historically pump near-event IV by 10-15 vol points.

---

## Step 5: IV vs Realized (IV-RV Spread)

Pull BTC daily closes for the past 30 days from any spot source (e.g. Coingecko, or use the Deribit index `/get_index_price` snapshot for now and an external history elsewhere). Compute realized vol:

```python
import numpy as np
def realized_vol(prices, window=30):
    logret = np.log(prices).diff()
    return logret.rolling(window).std() * np.sqrt(365)
```

Compare to DVOL:
```
iv_rv_spread = DVOL − realized_vol_30d
```

- Sustained `iv_rv_spread > 10` vol points → vol-sellers earn the variance risk premium
- `iv_rv_spread < 0` (realized > implied) → vol expansion regime, vol buyers winning

This is the canonical "vol carry" trade in crypto. Risk: realized can spike fast — funding stress, exchange collapses, hack rumors all blow it up. Size with `position-sizing`.

---

## Step 6: Respond to User

1. Current DVOL (BTC + ETH if relevant) + 1y percentile.
2. Regime classification.
3. IV-RV spread context.
4. Term-structure read (if user asks beyond DVOL alone).
5. One-line trade implication if a clear regime — but no naked recommendations without sizing constraints.

Hand off to:
- `deribit-options-chain` for the full surface
- `inverse-options-pricing` for theoretical contract prices
- `position-sizing` if user wants to size a vol-carry trade

---

## Caveats

- **DVOL update frequency**: Deribit publishes DVOL continuously; the index uses a 1-second tick. Polling once/minute is plenty.
- **DVOL ≠ ATM IV**: DVOL is a variance-swap replication, not the ATM 30-day IV. They typically differ by 1-3 vol points (DVOL is biased higher because of OTM contributions).
- **ETH DVOL liquidity**: Thinner than BTC. Daily-bar level is reliable; intraday can spike on a single block trade through an OTM wing.
- **No SOL DVOL** as of 2025-Q4 — too thin a chain to compute.
