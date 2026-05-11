# Inverse Options — Full Math

## Why Deribit Quotes in Coin

Margin and settlement are in BTC (or ETH) so that traders don't need fiat rails. A 1-BTC call on Deribit:
- Premium: paid in BTC
- Notional: 1 BTC
- Payoff at expiry: in BTC, equals `max(S_T − K, 0) / S_T`

The `/ S_T` divisor exists because the BTC delivered at expiry is worth `S_T` USD, so a USD payoff of `max(S_T − K, 0)` translates to `max(S_T − K, 0)/S_T` BTC.

## Pricing — Two Equivalent Views

### View 1: Vanilla price under USD numeraire, then convert

The USD-denominated value of the inverse option is the same as a regular call (the holder receives `max(S_T − K, 0)` USD worth of BTC):

```
V_usd(S, K, T, r, σ) = BS_call(S, K, T, r, σ)         [vanilla Black-Scholes]
V_coin(S, K, T, r, σ) = V_usd / S
```

### View 2: Risk-neutral pricing under coin numeraire

Under the BTC-numeraire measure, the drift of S becomes `r − q + σ²` (the +σ² is the Itô correction from switching numeraire 1/S). Then:

```
V_coin = E^Q_coin[ max(1 − K/S_T, 0) ]    (for a call)
       = N(d1') − (K/S)·e^(−r·T)·N(d2')
where d1' = (ln(S/K) + (r − q + σ²/2)·T) / (σ√T) − σ√T
       d2' = d1' − σ√T
```

For European-style options these two views give **identical numerical results**. Use View 1 — it's simpler.

## Greeks Derivations

With `V_coin = V_usd / S`:

```
∂V_coin/∂S = (V_usd' · S − V_usd) / S²            [quotient rule]
           = delta_usd/S − V_coin/S
           = (delta_usd − V_coin) / S
```

So:
```
delta_coin = (delta_usd − V_coin) / S
USD_equivalent_delta = delta_coin · S = delta_usd − V_coin
```

This is why Deribit's call delta near ATM is around 0.3-0.45 (not 0.5 like equity): the inverse adjustment subtracts the call's intrinsic-ish value.

### Vega
σ doesn't appear in the V_usd/S divisor, so:
```
vega_coin = vega_usd / S
```

### Theta
T doesn't appear in S either:
```
theta_coin = theta_usd / S
```
(plus a tiny correction if r ≠ 0)

### Gamma
Second derivative w.r.t. S:
```
∂²V_coin/∂S² = gamma_usd/S − 2·delta_usd/S² + 2·V_usd/S³
             = gamma_usd/S − 2·delta_usd/S² + 2·V_coin/S²
```

In practice, hedge using USD-equivalent gamma (= `gamma_usd`) and let your delta-hedging skill compute share-equivalents in USD terms — it's much easier.

## ETH Options — Staking Yield

ETH options on Deribit are settled in ETH. Holders of long-dated ETH options are NOT receiving staking yield (the writer keeps the ETH). To match the model to reality, use `q = staking_yield` (currently ≈ 3-4% annualized).

This makes calls cheaper and puts more expensive than the q = 0 case. Deribit's `mark_iv` is computed with `q = 0` in their public mark formula, so back-solved IVs may look slightly different if you include q.

For BTC, q = 0 (no native yield).

## Funding Rate vs r

Deribit historically used `r = 0` because USDT/BTC funding is not the same as a risk-free rate. For arbitrage-grade pricing you'd back out an "implied repo" from put-call parity on the chain:

```
C − P = (S·e^(−q·T) − K·e^(−r·T)) / S
       (in coin units, for inverse options)
```

Solve for r given (C, P, S, K, T, q) at the ATM strike — that's the "implied funding rate" the options market is using. Often very different from perp funding.

## DVOL Caveat

Deribit's DVOL is computed from the listed BTC/ETH option chain using a variance-swap-style replication, similar to VIX. It's a 30-day forward expected vol of **USD-denominated** returns. When you back out IVs from inverse-option mark prices, you get the same number — the σ is invariant under the numeraire change for European options.

So: DVOL ≈ ATM 30-day IV of BTC. Use DVOL as a regime indicator just like VIX.

## Common Mistakes

| Mistake | Fix |
|---|---|
| "Deribit delta of 0.4 means I need 0.4 BTC of hedge" | No — convert to USD-equivalent first (`delta_coin × S × contracts`). A 0.4 coin-delta call on 10 contracts at BTC=$100k needs ≈ $400k of spot or perp short. |
| "I'll use yfinance BS to price BTC options" | yfinance has no BTC options. Use Deribit. And remember to divide by S to get the coin price for comparison. |
| "Greeks are the same as equities" | No — divide by S. The coin-vs-USD delta gap is significant. |
| "I'll re-fit DVOL by hand" | DVOL is a published index — use `dvol-index` skill. Re-fitting is doable but never matches Deribit's published value because of strike grid weighting. |
