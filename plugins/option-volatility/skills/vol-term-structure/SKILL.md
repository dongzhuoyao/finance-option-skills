---
name: vol-term-structure
description: >
  Analyze the time-axis implied vol term structure for a ticker — ATM IV across expiries, contango
  vs backwardation, and event-vol jumps (earnings, FOMC, expirations). Use this skill when the user
  asks about IV across expiries, VIX9D/VIX/VIX3M shape, earnings vol bump, near-vs-far IV, or
  whether to roll or sell vol farther/nearer. Triggers: "term structure", "contango", "backwardation",
  "vol curve", "VIX9D vs VIX", "earnings vol crush", "IV rank by expiry", "near vol vs far vol".
  Default: ATM strike, all listed expiries up to 1Y.
---

# Vol Term Structure

Pull ATM IV across all listed expiries, identify contango vs backwardation, and flag event-driven jumps.

---

## Step 1: Pull All Expiries

Use `yfinance-options` to list expiries; filter to expiries with at least 10 contracts and a market-mid spread under 30% for the ATM strike. Keep up to 12 expiries (1Y horizon is plenty for vol-term-structure work).

---

## Step 2: Compute ATM IV per Expiry

For each expiry T:
- Forward F = S · e^((r−q)·T)
- ATM strike = strike closest to F. If the listed strike is > 1% from F, interpolate IV between the two bracketing strikes.
- Take the average of the ATM call IV and the ATM put IV. They should be very close (put-call parity in vol); a wide gap means stale quotes — flag it.

---

## Step 3: Classify Shape

```
slope_short = IV(2nd expiry) − IV(1st expiry)
slope_long  = IV(last expiry) − IV(median expiry)
```

| Shape | Pattern | Interpretation |
|---|---|---|
| **Contango** | Front-end IV < back-end IV (positive slope) | Quiet regime, vol-sellers' market |
| **Backwardation** | Front-end IV > back-end IV (negative slope) | Stress regime, vol-buyers' market |
| **Humped** | Local maximum at one expiry, then decline | Event vol — earnings, FDA, Fed |
| **Bowl** | Local minimum mid-curve | Rare; suggests pricing dislocation |

Print the percentile of `slope_short` over the past year. Sustained backwardation often precedes selloffs (or follows them — context matters).

---

## Step 4: Detect Event-Vol Jumps

If two adjacent expiries differ by > 3 vol points and there's a known scheduled event between them (earnings date, FOMC, ex-div), flag the event as the likely cause.

Check the earnings calendar:
```
!`python3 -c "import yfinance as yf; t = yf.Ticker('AAPL').calendar; print(t)" 2>/dev/null || echo "Calendar unavailable"`
```

The expiry **after** an earnings date typically has +5 to +15 vol points on single-name names. This vol crashes 30-60% the day after the print — the canonical "earnings vol crush" trade.

---

## Step 5: Term-Structure Trade Patterns

| Setup | Trade | Why |
|---|---|---|
| Steep contango, no event ahead | Calendar (sell near, buy far) | Front-month rolls down the curve |
| Backwardation extreme | Buy front-month vol | Curve mean-reverts to flat |
| Earnings bump | Sell front-of-earnings, buy back after | IV crush is reliable; magnitude isn't |
| Flat curve | No vol-curve trade — pick a different angle |

**Never** sell pre-event vol without a sizing plan. A miss-and-gap can erase weeks of premium collection.

---

## Step 6: Respond to User

1. The full term structure table (expiry, DTE, ATM IV, slope from previous).
2. Shape classification + percentile.
3. Any event-vol jumps flagged.
4. One trade idea if a clear pattern emerges.

For the cross-strike picture (skew at each expiry), recommend `vol-skew` or full `iv-surface`.
