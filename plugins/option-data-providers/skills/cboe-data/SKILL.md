---
name: cboe-data
description: >
  Read CBOE volatility indices and ratios for market-wide vol context — VIX, VVIX, SKEW, VIX9D, VIX3M,
  VIX6M, put/call ratio, and term-structure ratios like VIX9D/VIX (frontend stress) and VIX/VIX3M
  (curve shape). Use this skill when the user asks about market vol regime, complacency vs fear,
  term-structure signals, or wants context for individual-name trades. Triggers: "VIX", "VVIX",
  "SKEW", "put/call ratio", "vol regime", "is the market complacent", "frontend stress",
  "VIX term structure". Defaults: spot levels + 1y percentile.
---

# CBOE Volatility Indices Reader

Pull and interpret CBOE indices for market-vol regime context.

---

## Step 1: Pull the Index Snapshot

Via yfinance (free, delayed) or CBOE direct API.

```python
import yfinance as yf
INDICES = {
    'VIX':    '^VIX',    # 30-day expected SPX vol
    'VVIX':   '^VVIX',   # vol of VIX
    'SKEW':   '^SKEW',   # tail-risk premium
    'VIX9D':  '^VIX9D',  # 9-day expected vol
    'VIX3M':  '^VIX3M',  # 3-month expected vol
    'VIX6M':  '^VIX6M',  # 6-month expected vol
    'PCRATIO': None,     # CBOE publishes daily; not on yfinance
}
def snapshot():
    out = {}
    for name, sym in INDICES.items():
        if sym:
            t = yf.Ticker(sym)
            out[name] = t.fast_info['lastPrice']
    return out
```

If the snapshot fails for any index, **list which ones** — don't paper over with NaN.

---

## Step 2: Interpret Levels

| Index | Calm | Normal | Elevated | Stressed |
|---|---|---|---|---|
| **VIX** | < 12 | 12–18 | 18–25 | > 25 |
| **VVIX** | < 80 | 80–100 | 100–120 | > 120 |
| **SKEW** | < 120 | 120–135 | 135–150 | > 150 |
| **PCRATIO (equity)** | < 0.6 (greed) | 0.6–1.0 | 1.0–1.3 | > 1.3 (fear) |

Levels alone aren't enough — show 1y percentile too.

---

## Step 3: Term-Structure Ratios

```
ratio_9d_30d = VIX9D / VIX
ratio_30d_3m = VIX / VIX3M
ratio_30d_6m = VIX / VIX6M
```

| Ratio | < 1 | > 1 |
|---|---|---|
| VIX9D / VIX | Curve normal (contango) | Frontend stress (backwardation) |
| VIX / VIX3M | Front below back (calm) | Front above back (stress) |

Sustained `VIX > VIX3M` for several days is the canonical "buy SPX" signal at extremes (mean reversion of curve). Sustained `VIX9D / VIX > 1` precedes vol expansion more often than not.

---

## Step 4: Cross-Index Signals

| Signal | Pattern | Read |
|---|---|---|
| **Complacency** | VIX low, SKEW high, P/C low | Tail risk priced but spot vol cheap — gap risk |
| **Fear bid** | VIX high, VVIX high, SKEW high | Generalized stress, vol-of-vol elevated |
| **Vol crush ahead** | VIX9D / VIX > 1.1, no known event | Mean reversion likely; consider selling front-end vol *with* a stop |
| **Earnings season** | VIX > VIX3M but only on event-heavy weeks | Don't read as market-wide stress |

---

## Step 5: Respond to User

1. Snapshot table (index, level, 1y percentile, classification).
2. Term-structure ratios.
3. One-line regime call (complacent / normal / stressed / event-driven).
4. Trade ideas only if a strong signal — be specific about which expiry / which underlying. No generic "consider buying puts".

Recommend `vol-term-structure` to drill into a specific name, or `vol-skew` for per-name skew.

---

## Caveats

- VIX is **30-day forward**, model-derived from SPX option mids — it's not "what vol is now", it's "what option prices imply about the next 30 days".
- VVIX is vol of VIX; spikes when traders gamma-hedge VIX options.
- SKEW is non-trivial to interpret in absolute terms — its 1y percentile is more useful than the raw number.
- yfinance VIX data can be ~15min delayed. For real-time use CBOE's paid feed.
