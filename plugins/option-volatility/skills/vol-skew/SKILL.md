---
name: vol-skew
description: >
  Analyze the strike-axis implied volatility skew for a ticker — 25-delta risk reversal,
  butterfly, put-call skew, and skew steepness percentile. Use this skill when the user asks about
  put/call skew, downside protection cost, "is the skew rich", "compare AAPL skew to history",
  or directional implications of skew shape. Triggers: "vol skew", "smile", "risk reversal", "25d RR",
  "25d butterfly", "put skew rich/cheap", "tail risk premium", "skew percentile".
  Use even with partial input — default expiry: 30 DTE.
---

# Vol Skew Analysis

Quantify the strike-axis skew at a fixed expiry and put it in historical context.

---

## Step 1: Pull the Chain at Target Expiry

Use `yfinance-options`. Pick the expiry closest to user's requested DTE (default 30). Both puts and calls.

Compute the **forward**: `F = S · e^((r−q)·T)`. All skew metrics use moneyness w.r.t. F, not S.

---

## Step 2: Find 25-Delta Strikes

For each side (put and call), interpolate to find the strike where |Δ| = 0.25. Use linear interpolation on (Δ, IV) since both are smooth in K.

If 25-delta strikes are outside the listed range, **stop** and report — your chain isn't wide enough.

---

## Step 3: Skew Metrics

| Metric | Formula | Interpretation |
|---|---|---|
| **25-delta risk reversal** | IV(25d put) − IV(25d call) | + → put skew (downside fear). Equities ≈ +1 to +6 vol pts |
| **25-delta butterfly** | (IV(25d put) + IV(25d call))/2 − IV(50d ATM) | + → smile (kurtosis premium) |
| **Put-call skew slope** | (IV(90% moneyness) − IV(110% moneyness)) / 0.2 | per 1.0 of moneyness shift |
| **ATM IV** | IV at K = F | baseline level |

---

## Step 4: Historical Percentile

For each metric, compute its rolling 1-year percentile to flag "skew rich" (high pct, downside expensive) vs "skew cheap" (low pct, downside cheap).

If 1y of data isn't available (new IPO, etc.), use what you have and label the percentile window in the output.

---

## Step 5: Directional Interpretation

| Pattern | Meaning |
|---|---|
| Rising RR pct (e.g., +95th) | Puts richening — fear bid. Often coincident with VIX up. |
| Falling RR pct (e.g., +5th) | Puts cheapening — complacency. Calls may richen → speculative rally. |
| Rising butterfly | Tails get richer relative to body — jump expected (binary event, earnings). |
| Falling butterfly | Tails fade — quiet regime. |

**Single-name vs index**: index skew is structural (put hedging demand). Single-name skew is more directional. Don't apply SPX skew norms to TSLA.

---

## Step 6: Respond to User

Surface:
1. Current RR_25d, BF_25d, ATM IV, in vol points.
2. Percentile of each vs trailing 1y.
3. One-line trade idea if skew is at an extreme percentile (e.g., "RR at +97th pct → consider selling the 25-delta put if your view is bullish, since the put is rich vs history").
4. Caveat: skew can stay rich for a long time. Mean reversion is NOT a strategy by itself.

For full surface visualization, recommend `iv-surface`. For implementation of a skew trade, recommend `strategy-selector` → `options-payoff`.
