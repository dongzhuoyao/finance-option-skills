---
name: inverse-options-pricing
description: >
  Price coin-settled (inverse) options the way Deribit quotes them — premium and payoff in BTC/ETH,
  not USD. Use this skill whenever the user asks about Deribit option pricing, why Deribit's deltas
  differ from textbook BSM, or how to convert between coin-quoted and USD-quoted Greeks. Triggers:
  "inverse option", "coin-settled", "Deribit pricing", "why is Deribit delta different",
  "convert Deribit Greeks to USD", "BTC denominated payoff", "premium in BTC".
  Default: r = 0 (Deribit uses 0 risk-free historically), q = 0.
---

# Inverse (Coin-Settled) Options Pricing

Deribit BTC/ETH options are inverse-settled: 1 contract has a notional of **1 BTC (or 1 ETH)**, premium is quoted in coin, payoff at expiry is in coin. This changes the Greeks.

---

## Step 1: Understand the Payoff

For a Deribit BTC call with strike K, at expiry with index price S_T:

```
payoff_in_BTC = max(S_T − K, 0) / S_T       (call)
payoff_in_BTC = max(K − S_T, 0) / S_T       (put)
```

The `/ S_T` divisor is the inverse-settlement twist. As S_T rises, the call payoff in BTC **grows slower** than a USD call (and the BTC-denominated value is bounded by 1 BTC even as S_T → ∞).

USD payoff (for comparison): `payoff_in_USD = max(S_T − K, 0)` exactly as a vanilla call.

The two are related by:
```
coin_payoff = USD_payoff / S_T
```

---

## Step 2: Pricing Formula

Two equivalent ways to think about it:

### Method A — Price in USD, convert to coin

1. Compute the vanilla USD BS price `C_usd(S, K, T, r, σ)`.
2. Divide by spot: `C_coin = C_usd / S`.

This is what Deribit's `mark_price` field returns.

### Method B — Risk-neutral coin-numeraire

Under the coin-as-numeraire measure, the drift of S becomes `r − q + σ²` (the additional σ² is the inverse-numeraire correction). The pricing formula has different d1/d2 inputs.

For practical work, **Method A is simpler and gives identical results** for European inverse options. Use it.

---

## Step 3: Inputs

| Field | Notes | Default |
|---|---|---|
| S | Spot (BTC index = btc_usd) | required |
| K | Strike (USD) | required |
| T | Time to expiry in years (DTE/365) | required |
| r | Risk-free; Deribit historically uses 0 | 0 |
| q | Dividend / staking yield (BTC = 0; ETH staking ≈ 3-4%) | 0 |
| σ | IV (decimal) | from Deribit `mark_iv` |
| type | call or put | required |

---

## Step 4: Compute (Python)

```python
from math import log, sqrt, exp, pi
from scipy.stats import norm

def usd_bs_price(S, K, T, r, sigma, q=0, opt='call'):
    if T <= 0 or sigma <= 0:
        return max(S - K, 0) if opt == 'call' else max(K - S, 0)
    d1 = (log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*sqrt(T))
    d2 = d1 - sigma*sqrt(T)
    if opt == 'call':
        return S*exp(-q*T)*norm.cdf(d1) - K*exp(-r*T)*norm.cdf(d2)
    return K*exp(-r*T)*norm.cdf(-d2) - S*exp(-q*T)*norm.cdf(-d1)

def deribit_inverse_price(S, K, T, r, sigma, q=0, opt='call'):
    """Returns premium in coin units (BTC for BTC options)."""
    return usd_bs_price(S, K, T, r, sigma, q, opt) / S
```

To compare against Deribit's `mark_price`, this should match within rounding.

---

## Step 5: Inverse Greeks (the trick)

Deribit returns Greeks in **coin-per-coin-of-underlying-move** units. Conversions:

```
delta_coin = delta_usd / S − usd_price / S²
           ≈ delta_usd / S      (when usd_price ≪ S, deep OTM or near expiry)
```

More precisely, with `V_coin = V_usd / S`:

```
delta_coin = ∂V_coin / ∂S = (S · ∂V_usd/∂S − V_usd) / S²
           = (delta_usd · S − V_usd) / S²
           = delta_usd / S − V_usd / S²
```

For practical use:
- **USD-equivalent delta** = `delta_coin × S` (this is "how many USD do I make per $1 of spot move")
- **Coin notional**: 1 contract = 1 BTC, so coin delta of 0.5 means a $1 spot move changes the position by 0.5 coin × (1 / S_change) ≈ 0.5/S BTC. Confused yet? **Always reconvert to USD before reasoning about hedge ratios.**

### Vega
Coin vega: `vega_coin = vega_usd / S`. Convert with `× 1/100` for "per vol point" the same way.

### Theta
Coin theta: `theta_coin = theta_usd / S` (assuming S doesn't drift — small approximation).

### Gamma
Coin gamma is a mess of cross-terms. Use the USD price's gamma and convert with `gamma_coin ≈ gamma_usd / S − 2·delta_usd / S² + 2·V_usd / S³`. Or just use the USD Greeks and remember that hedge ratios should be in USD-equivalent terms.

---

## Step 6: Sanity Checks

1. **Put-call parity (inverse form)**: `C_coin − P_coin = 1 − K·e^(−rT)/S`. If r = 0: `C − P = 1 − K/S`. Verify within 1e-5.
2. **Bound**: `C_coin ≤ 1` always (you can't make more than 1 coin from a 1-coin call). `P_coin ≤ K·e^(−rT)/S`.
3. **Deep ITM call**: `C_coin → 1 − K·e^(−rT)/S` as S → ∞.
4. **Deep OTM call**: `C_coin → 0`.

---

## Step 7: Respond to User

1. Coin-quoted price (matches Deribit's `mark_price`).
2. USD-equivalent price = `coin_price × S`.
3. Greeks: report BOTH coin-denominated and USD-equivalent, with labels.
4. Comparison against the live Deribit `mark_iv` / `mark_price` if the user gave a contract name.

For full surface fitting, recommend `iv-surface` and tell the user the SVI fit is for vanilla — inverse-option vols can be back-solved to USD vols using the same σ (the IV is the same under both numeraires for European options, by the volatility invariance of measure change).

---

## Reference Files

- `references/inverse_math.md` — Full derivation of inverse Greeks, USD numeraire vs coin numeraire, ETH staking yield correction
