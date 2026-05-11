---
name: position-sizing
description: >
  Size an options position using Kelly-fraction or max-loss-based methods. Use this skill when the
  user asks "how many contracts should I sell/buy", "what size for this credit spread", or wants a
  capped-loss sizing for premium-selling strategies (cash-secured puts, credit spreads, iron condors).
  Triggers: "how much should I size", "kelly fraction", "position size", "max loss sizing", "risk per trade",
  "capital allocation for options".
  Do NOT use to recommend a trade — pair with `strategy-selector` for that. This skill only sizes a trade
  the user has already chosen.
---

# Options Position Sizing

Translate "I have $X in capital and a max-loss tolerance of $Y per trade" into a number of contracts. Supports two methods: **fixed-max-loss** (recommended for most retail) and **fractional Kelly** (for users who have a win-rate / payoff estimate).

---

## Step 1: Capture Inputs

| Input | Notes | Default |
|---|---|---|
| Total account capital | Used for the % of book check | required |
| Max loss tolerance per trade | $ or % of book | 1–3% of book |
| Trade max loss (per contract) | From `options-payoff` | required |
| Trade max profit (per contract) | required for Kelly | |
| Win probability estimate | required for Kelly | |
| Buying-power requirement (per contract) | For broker margin sizing | varies |

If trade max loss is "unbounded" (naked call, ratio spread short side) → **refuse to size** with this skill. Use a closing rule instead, or pick a different structure via `strategy-selector`.

---

## Step 2: Fixed-Max-Loss Sizing

```
N_contracts = floor( max_loss_tolerance / per_contract_max_loss )
```

Cap further by:
1. Buying-power: `N ≤ floor(account_capital / BP_per_contract)`
2. Concentration: a single underlying shouldn't be > 20–25% of book BP

Worked example: $50,000 account, willing to risk $500/trade, iron condor with $4.20 max loss → N = floor(500 / 420) = 1 contract.

---

## Step 3: Fractional Kelly (advanced)

Full Kelly fraction:
```
f* = (p · b − (1 − p)) / b
```
Where p = win prob, b = win/loss ratio = max_profit / max_loss.

**Do not use full Kelly for options.** Standard practice is `0.25 · f*` ("quarter Kelly") because:
1. Your p estimate is noisy.
2. Path-dependent drawdowns are brutal at full Kelly.
3. Variance of returns is enormous — full Kelly leads to expected log-utility but huge equity swings.

```
contracts = floor( 0.25 · f* · account_capital / per_contract_max_loss )
```

Refuse to size if f* ≤ 0 (negative edge — the trade has negative expected value). State this explicitly. Don't size a losing trade.

---

## Step 4: Sanity Checks

1. **Won't blow the account on N consecutive losses**: at 3 standard deviations of consecutive losses (binomial), would the account survive? If not, halve N.
2. **Margin call risk on undefined-risk legs**: if buying-power requirement could grow with adverse moves (short put), assume worst case = strike × 100. Use that as your BP, not the broker's initial margin.
3. **Correlated positions**: if user already has 5 short-vol trades on tech names, don't size a 6th tech short-vol — concentration risk dominates per-trade math.

---

## Step 5: Respond to User

| Method | N contracts | $ at risk | % of book | Notes |
|---|---|---|---|---|
| Fixed max-loss | … | … | … | Recommended |
| Quarter Kelly | … | … | … | Requires reliable p estimate |
| BP-capped | … | … | … | Broker constraint |

End with: "**Final size: N = min(...)**" — the most restrictive of the three.

Important caveat to include: "Your stated win probability is a guess. If it's wrong by 10%, Kelly sizing scales linearly. Always start smaller than you think."
