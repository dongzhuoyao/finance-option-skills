# Black-Scholes-Merton Reference

## Full Closed-Form (with continuous dividend yield q)

```
d1 = [ ln(S/K) + (r − q + σ²/2)·T ] / (σ·√T)
d2 = d1 − σ·√T

C = S·e^(−qT)·N(d1) − K·e^(−rT)·N(d2)
P = K·e^(−rT)·N(−d2) − S·e^(−qT)·N(−d1)
```

## Put-Call Parity (with dividends)

```
C − P = S·e^(−qT) − K·e^(−rT)
```

For non-dividend stocks (q = 0) this simplifies to `C − P = S − K·e^(−rT)`.

## Python Implementation

```python
from math import log, sqrt, exp
from scipy.stats import norm

def bs_price(S, K, T, r, sigma, q=0.0, opt_type='call'):
    if T <= 0 or sigma <= 0:
        intrinsic = max(S - K, 0) if opt_type == 'call' else max(K - S, 0)
        return intrinsic
    d1 = (log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*sqrt(T))
    d2 = d1 - sigma*sqrt(T)
    if opt_type == 'call':
        return S*exp(-q*T)*norm.cdf(d1) - K*exp(-r*T)*norm.cdf(d2)
    return K*exp(-r*T)*norm.cdf(-d2) - S*exp(-q*T)*norm.cdf(-d1)
```

## JavaScript Implementation (no scipy)

```js
// Abramowitz & Stegun 26.2.17 approximation, ~1e-7 accuracy
function normCDF(x) {
  const a1 =  0.254829592, a2 = -0.284496736, a3 =  1.421413741;
  const a4 = -1.453152027, a5 =  1.061405429, p  =  0.3275911;
  const sign = x < 0 ? -1 : 1;
  const ax = Math.abs(x) / Math.SQRT2;
  const t = 1.0 / (1.0 + p*ax);
  const y = 1.0 - (((((a5*t + a4)*t) + a3)*t + a2)*t + a1)*t * Math.exp(-ax*ax);
  return 0.5 * (1.0 + sign*y);
}

function bsPrice(S, K, T, r, sigma, q = 0, type = 'call') {
  if (T <= 0 || sigma <= 0) {
    return type === 'call' ? Math.max(S - K, 0) : Math.max(K - S, 0);
  }
  const d1 = (Math.log(S/K) + (r - q + 0.5*sigma*sigma)*T) / (sigma*Math.sqrt(T));
  const d2 = d1 - sigma*Math.sqrt(T);
  if (type === 'call') {
    return S*Math.exp(-q*T)*normCDF(d1) - K*Math.exp(-r*T)*normCDF(d2);
  }
  return K*Math.exp(-r*T)*normCDF(-d2) - S*Math.exp(-q*T)*normCDF(-d1);
}
```

## Discrete Dividends

Black-Scholes assumes continuous dividend yield. For discrete cash dividends D_i paid at times t_i ≤ T, replace S with the "escrowed" spot:

```
S* = S − Σ_i D_i · e^(−r·t_i)
```

Then price with S* and q = 0. This is the "escrowed dividend" adjustment; it slightly underestimates puts on dividend-paying names.

## Bounds (Arbitrage Limits)

For a non-dividend call:
```
max(S − K·e^(−rT), 0) ≤ C ≤ S
```

With continuous dividend q:
```
max(S·e^(−qT) − K·e^(−rT), 0) ≤ C ≤ S·e^(−qT)
```

Put bounds (with dividends):
```
max(K·e^(−rT) − S·e^(−qT), 0) ≤ P ≤ K·e^(−rT)
```

If model price violates the lower bound, inputs are inconsistent — do not coerce. Surface the violation.

## Batch Pricing Template (Python)

```python
import numpy as np
from scipy.stats import norm

def bs_price_batch(S, K, T, r, sigma, q=0.0, opt_type='call'):
    S, K, T, sigma = map(np.asarray, (S, K, T, sigma))
    safe = (T > 0) & (sigma > 0)
    out = np.where(opt_type == 'call', np.maximum(S - K, 0.0), np.maximum(K - S, 0.0))
    if not safe.any():
        return out
    d1 = np.where(safe, (np.log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*np.sqrt(T)), 0.0)
    d2 = d1 - sigma*np.sqrt(T)
    if opt_type == 'call':
        priced = S*np.exp(-q*T)*norm.cdf(d1) - K*np.exp(-r*T)*norm.cdf(d2)
    else:
        priced = K*np.exp(-r*T)*norm.cdf(-d2) - S*np.exp(-q*T)*norm.cdf(-d1)
    return np.where(safe, priced, out)
```
