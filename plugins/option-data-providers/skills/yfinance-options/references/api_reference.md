# yfinance Options — API Reference

## Full Template (with all filters)

```python
import yfinance as yf
import pandas as pd
import numpy as np
from datetime import datetime

def get_full_chain(ticker_sym, expiries=None, drop_bad=True):
    """Pull all (or selected) expiries, concat, and tag with metadata."""
    t = yf.Ticker(ticker_sym)
    spot = t.fast_info['lastPrice']
    if expiries is None:
        expiries = list(t.options)
    rows = []
    for exp in expiries:
        try:
            ch = t.option_chain(exp)
        except Exception as e:
            print(f"WARN: {exp} failed: {e}")  # don't silently skip
            continue
        for side, df in [('call', ch.calls), ('put', ch.puts)]:
            df = df.copy()
            df['type'] = side
            df['expiry'] = exp
            rows.append(df)
    if not rows:
        raise RuntimeError(f"No chains pulled for {ticker_sym}")
    df = pd.concat(rows, ignore_index=True)
    df['mid'] = (df['bid'] + df['ask']) / 2
    df['spread'] = df['ask'] - df['bid']
    df['spread_pct'] = np.where(df['mid'] > 0, df['spread'] / df['mid'], np.nan)
    df['DTE'] = (pd.to_datetime(df['expiry']) - pd.Timestamp.today().normalize()).dt.days
    df['T'] = df['DTE'] / 365.0
    df['moneyness'] = df['strike'] / spot

    if drop_bad:
        before = len(df)
        df = df[(df['bid'] > 0) & (df['ask'] > 0)
                & (df['spread_pct'] < 0.5)
                & (df['mid'] >= 0.05)
                & ((df['volume'].fillna(0) > 0) | (df['openInterest'].fillna(0) >= 10))]
        df = df.reset_index(drop=True)
        print(f"Filtered {before - len(df)}/{before} contracts on quality")

    return {'spot': spot, 'as_of': datetime.utcnow().isoformat(), 'chain': df}
```

## IV Recomputation (when yfinance IV is stale)

```python
from math import log, sqrt, exp, pi
from scipy.stats import norm

def implied_vol_newton(price, S, K, T, r, opt_type='call', q=0.0, sigma0=0.3, tol=1e-6, max_iter=50):
    sigma = sigma0
    for _ in range(max_iter):
        d1 = (log(S/K) + (r - q + 0.5*sigma**2)*T) / (sigma*sqrt(T))
        d2 = d1 - sigma*sqrt(T)
        pdf = (1/sqrt(2*pi))*exp(-0.5*d1*d1)
        vega = S*exp(-q*T)*pdf*sqrt(T)
        if opt_type == 'call':
            model = S*exp(-q*T)*norm.cdf(d1) - K*exp(-r*T)*norm.cdf(d2)
        else:
            model = K*exp(-r*T)*norm.cdf(-d2) - S*exp(-q*T)*norm.cdf(-d1)
        if vega < 1e-10:
            return None
        diff = model - price
        if abs(diff) < tol:
            return sigma
        sigma -= diff / vega
        if sigma <= 0:
            sigma = 0.01
    return None

def add_iv_column(df, spot, r=0.043, q=0.0):
    ivs = []
    for _, row in df.iterrows():
        iv = implied_vol_newton(row['mid'], spot, row['strike'], row['T'], r, row['type'], q)
        ivs.append(iv)
    df['iv_recomputed'] = ivs
    return df
```

## Rate-limit Handling

yfinance backs onto Yahoo's free endpoint. Heavy chains (full QQQ across 30 expiries) can hit 429s.

```python
import time
def retry_chain(ticker, expiry, attempts=3, base_delay=1.0):
    for i in range(attempts):
        try:
            return yf.Ticker(ticker).option_chain(expiry)
        except Exception as e:
            if i == attempts - 1:
                raise
            time.sleep(base_delay * (2 ** i))
```

## Caveats

- **Tail recursion on stale cache**: yfinance caches the chain object per-process. To force refresh, instantiate a new `Ticker` object.
- **Index options (SPX, NDX, RUT)**: ticker symbols `^SPX`, `^NDX`, `^RUT`. European-style, cash-settled.
- **VIX options**: ticker `^VIX`. European-style, but settle to a special open auction (SOQ), not the displayed VIX.
- **Adjusted strikes after splits/specials**: yfinance does NOT always adjust strikes. Check `Ticker.actions` for recent corporate events.
