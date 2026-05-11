---
name: gex-max-pain
description: >
  Compute dealer Gamma Exposure (GEX) and Max Pain strike for BTC or ETH at any Deribit expiry.
  Use this skill when the user asks about dealer positioning, "magnet" strike for expiry,
  gamma walls, or expiry-related spot pinning. Triggers: "GEX", "gamma exposure", "max pain",
  "BTC expiry magnet", "where will BTC pin on Friday", "dealer gamma BTC", "options expiry impact".
  Defaults: BTC, nearest Friday expiry, all strikes with open interest above zero.
---

# GEX and Max Pain (Deribit)

Two heuristics popular in crypto options:
- **Max Pain**: the strike at which the most options expire worthless (sum of ITM put + call OI is minimized). Folk theory: spot is "pulled" toward this strike near expiry.
- **GEX**: sum of dealer gamma exposure across the chain. Positive GEX = dealers are long gamma and will dampen vol via hedging flows. Negative GEX = dealers short gamma, hedging amplifies moves.

Both are *positioning indicators*, not forecasts. Don't treat them as price targets.

---

## Step 1: Pull the Chain at Target Expiry

Use `deribit-options-chain` to get the full chain. Filter to a single `expiry` (default: nearest Friday).

Required columns per strike + side: `strike`, `option_type` (C/P), `open_interest` (contracts), `mark_iv`, `gamma` (from Deribit ticker, coin-units).

If gamma isn't in the chain summary, compute it via `inverse-options-pricing` using `mark_iv` and `underlying_price`. Don't silently skip the conversion — record which gammas came from where.

---

## Step 2: Max Pain Calculation

For each candidate spot S_test (typically every $500 strike for BTC, every $25 for ETH):

```python
import pandas as pd
def max_pain(chain_at_expiry):
    """chain has columns: strike, option_type ('C'/'P'), open_interest"""
    strikes = sorted(chain_at_expiry['strike'].unique())
    losses = []
    for S in strikes:
        # Holder P&L at expiry (ignoring premium — pure intrinsic)
        c_oi = chain_at_expiry[chain_at_expiry['option_type'] == 'C']
        p_oi = chain_at_expiry[chain_at_expiry['option_type'] == 'P']
        call_loss = ((S - c_oi['strike']).clip(lower=0) * c_oi['open_interest']).sum()
        put_loss  = ((p_oi['strike'] - S).clip(lower=0) * p_oi['open_interest']).sum()
        losses.append({'spot': S, 'total_holder_payoff': call_loss + put_loss})
    df = pd.DataFrame(losses)
    return df.loc[df['total_holder_payoff'].idxmin(), 'spot']
```

Report the max-pain strike AND a chart of the holder-payoff curve so the user sees how flat/peaked it is.

---

## Step 3: GEX Calculation

Assumption: dealers are **long calls and short puts** on net (the canonical dealer positioning in equity index options). Crypto often inverts this — retail buys upside calls heavily, so dealers are net SHORT calls in BTC. Verify with order-flow data if possible, but for a first cut:

```python
def gex(chain_at_expiry, spot, multiplier=100_000):
    """Returns GEX in USD per 1% spot move squared.
       multiplier scales gamma_per_share to per-position-USD."""
    # Convention: + GEX = dealer long gamma (vol-suppressing)
    # Assume dealers are short calls and long puts → flip signs
    c = chain_at_expiry[chain_at_expiry['option_type'] == 'C']
    p = chain_at_expiry[chain_at_expiry['option_type'] == 'P']
    # gamma_usd ≈ gamma_coin * S (rough — see inverse-options-pricing)
    g_call = (c['gamma_usd'] * c['open_interest']).sum()
    g_put  = (p['gamma_usd'] * p['open_interest']).sum()
    # short calls = dealer short gamma there; long puts = dealer long gamma there
    gex_total = (-g_call + g_put) * spot * multiplier / 100
    return gex_total
```

**Critical caveat**: dealer positioning is unknowable without flow data. The "dealers short calls in crypto" assumption is a heuristic — calibrate sign by checking actual flow if SpotGamma/Genesis Vol/Greeks.live data is available. Without flow data, report GEX magnitude but be explicit that the sign is assumed.

---

## Step 4: Gamma Profile by Strike

Plot net gamma exposure per strike. The largest spikes are the "gamma walls" — strikes where dealer hedging is most active.

```python
def gamma_by_strike(chain_at_expiry, spot):
    g = chain_at_expiry.copy()
    g['signed_gamma'] = (g.apply(lambda r: r['gamma_usd'] * r['open_interest']
                                  * (-1 if r['option_type']=='C' else 1), axis=1))
    return g.groupby('strike')['signed_gamma'].sum().reset_index()
```

Render as a horizontal bar chart with strike on y-axis. Annotate the max-pain strike and current spot.

---

## Step 5: Interpret

| Pattern | Read |
|---|---|
| Max pain near spot, low OI | Weak magnet — don't expect pinning |
| Max pain near spot, high OI | Plausible magnet effect into expiry |
| Spot far from max pain | Either dealers will hedge into max pain (theory) OR price has run too far (more likely in trend regimes) |
| Large positive GEX above spot | Resistance — dealers sell into rallies |
| Large negative GEX below spot | Acceleration zone — dealers sell into selloffs (gamma squeeze down) |
| 0DTE expiry, high OI | Strong pinning candidate at the nearest big-OI strike |

---

## Step 6: Respond to User

1. Expiry chosen + DTE.
2. Max-pain strike + holder-payoff chart.
3. Current GEX magnitude (with sign caveat).
4. Top 3 gamma walls (by absolute exposure).
5. Spot's position relative to walls.
6. One-line interpretation with appropriate humility ("max pain is a folk indicator — useful in low-vol regimes, broken during trends").

Hand off to:
- `options-payoff` if user wants to position around a wall
- `delta-hedging` for actively managing exposure through expiry
- `dvol-index` for vol regime context

---

## What This Skill Won't Do

- Predict tomorrow's price. GEX/max pain are positioning maps, not forecasts.
- Pretend dealer positioning is known. Without flow data, signs are estimated.
- Run on illiquid expiries — if total OI across the expiry < 200 contracts, **stop** and tell the user the data is too thin to mean anything.
