# Black-Scholes JS Implementation (drop-in for widgets)

```js
// Abramowitz & Stegun 26.2.17 — ~1e-7 accuracy
function normCDF(x) {
  const a1 =  0.254829592, a2 = -0.284496736, a3 =  1.421413741;
  const a4 = -1.453152027, a5 =  1.061405429, p  =  0.3275911;
  const sign = x < 0 ? -1 : 1;
  const ax = Math.abs(x) / Math.SQRT2;
  const t = 1.0 / (1.0 + p*ax);
  const y = 1.0 - (((((a5*t + a4)*t) + a3)*t + a2)*t + a1)*t * Math.exp(-ax*ax);
  return 0.5 * (1.0 + sign*y);
}

function bsCall(S, K, T, r, sigma, q = 0) {
  if (T <= 0 || sigma <= 0) return Math.max(S - K, 0);
  const sqrtT = Math.sqrt(T);
  const d1 = (Math.log(S/K) + (r - q + 0.5*sigma*sigma)*T) / (sigma*sqrtT);
  const d2 = d1 - sigma*sqrtT;
  return S*Math.exp(-q*T)*normCDF(d1) - K*Math.exp(-r*T)*normCDF(d2);
}

function bsPut(S, K, T, r, sigma, q = 0) {
  if (T <= 0 || sigma <= 0) return Math.max(K - S, 0);
  const sqrtT = Math.sqrt(T);
  const d1 = (Math.log(S/K) + (r - q + 0.5*sigma*sigma)*T) / (sigma*sqrtT);
  const d2 = d1 - sigma*sqrtT;
  return K*Math.exp(-r*T)*normCDF(-d2) - S*Math.exp(-q*T)*normCDF(-d1);
}
```

## Convention Reminders

- Inputs in **annual** units: T in years, sigma as decimal (0.20 = 20%), r as decimal (0.043 = 4.3%).
- `sigma * 100` when displaying as a percent; user-facing sliders should label "IV %".
- For weeklies / 0DTE: T = DTE / 365 (don't switch to business-day calendars unless explicit).
- For SPX / NDX: cash-settled European; no dividend yield needed on the index itself (use q=0).
- For SPY / QQQ ETFs: discrete dividends → use yfinance `dividends` and adjust S via PV-of-dividends trick.
