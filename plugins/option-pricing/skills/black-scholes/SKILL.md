---
name: black-scholes
description: >
  Price European call and put options using the Black-Scholes-Merton closed-form model.
  Use this skill whenever the user asks for a theoretical option price, a "fair value" estimate,
  put-call parity check, or wants to compare a quoted premium to a model price. Triggers include:
  any mention of Black-Scholes, BSM, theoretical price, fair value, intrinsic vs extrinsic,
  put-call parity, continuous dividend yield, risk-free rate, or phrases like
  "what should this option be worth", "is this overpriced", "price this call", "value this put".
  Activate even with partial input — use sensible defaults (r = 4.3%, q = 0, T = 30/365).
---

# Black-Scholes Option Pricing

Compute the theoretical price of a European call or put under the Black-Scholes-Merton model with optional continuous dividend yield.

The model assumes log-normal underlying dynamics, constant volatility and rates, no early exercise, no transaction costs, and continuous trading.

---

## Step 1: Extract Inputs

| Field | Notes | Default if missing |
|---|---|---|
| S — spot price | Current underlying price (NOT strike) | Required — ask user if missing |
| K — strike | The contract strike | Required |
| T — time to expiry | In **years** (DTE / 365 for calendar days) | 30/365 |
| r — risk-free rate | Annualized, continuous compounding | 0.043 |
| q — dividend yield | Continuous; 0 for non-dividend names | 0 |
| σ — volatility | Annualized standard deviation of log-returns | 0.20 if no quote |
| type | "call" or "put" | Required |

**Critical**: σ is annualized. If user says "20% IV" that means σ = 0.20, not 20.

**Current risk-free reference (3M T-bill):**
```
!`python3 -c "import yfinance as yf; r = yf.Ticker('^IRX').fast_info['lastPrice']/100; print(f'r ≈ {r:.4f}')" 2>/dev/null || echo "r unavailable — default to 0.043"`
```

---

## Step 2: Compute d1, d2

```
d1 = [ ln(S/K) + (r − q + σ²/2)·T ] / (σ·√T)
d2 = d1 − σ·√T
```

Use `scipy.stats.norm.cdf` for N(·). If scipy is unavailable, use the Horner approximation in `references/bs_formulas.md`.

---

## Step 3: Compute Price

**Call:**
```
C = S·e^(−qT)·N(d1) − K·e^(−rT)·N(d2)
```

**Put:**
```
P = K·e^(−rT)·N(−d2) − S·e^(−qT)·N(−d1)
```

**Edge cases — handle explicitly, never silently:**
- T ≤ 0 → return intrinsic: max(S−K, 0) for call, max(K−S, 0) for put. Print a warning.
- σ ≤ 0 → BS undefined. Return intrinsic and warn.
- S, K ≤ 0 → reject input with a clear error.

---

## Step 4: Sanity Checks

Run BOTH checks and report the result before quoting the price:

1. **Bounds**: A call must satisfy `max(0, S·e^(−qT) − K·e^(−rT)) ≤ C ≤ S·e^(−qT)`. Same shape for puts. If violated, the inputs are inconsistent.
2. **Put-call parity**: `C − P = S·e^(−qT) − K·e^(−rT)`. Price both legs from the same σ; the residual must be ≤ 1e-6 in absolute terms.

If either fails, **stop and report the failure with the residual** — do not "round to fix it."

---

## Step 5: Respond to User

Report:
1. The theoretical price (round to 4 decimals for premium, 2 decimals if > $10).
2. Intrinsic vs. extrinsic split: `intrinsic = max(S−K, 0)` (call), extrinsic = price − intrinsic.
3. The exact assumption set used (S, K, T, r, q, σ).
4. A one-line comparison to the user's market quote if provided ("model 4.12 vs quoted 4.35 → 5.6% rich").

For deeper diagnostics (e.g., why model and market disagree), suggest the `greeks-calculator` or `iv-surface` skills.

---

## Reference Files

- `references/bs_formulas.md` — Full derivation, scipy-free normCDF, dividend handling, batch pricing template
