---
name: binomial-pricing
description: >
  Price American options and capture the early-exercise premium via a Cox-Ross-Rubinstein binomial tree.
  Use this skill whenever the user asks about American-style options, dividend-paying stocks with optimal exercise,
  early exercise decisions, or compares American vs European pricing. Triggers: "American option",
  "early exercise", "dividend exercise", "exercise premium", "CRR tree", "binomial", "lattice",
  "should I exercise this option early", "American put on TSLA", "is it ever optimal to exercise".
  Activate even with partial input — defaults: r = 4.3%, q = 0, steps = 200.
---

# Binomial Tree Pricing (CRR)

Price American calls and puts via a Cox-Ross-Rubinstein binomial tree. Recover the European price as a sanity check and report the **early-exercise premium** = American − European.

---

## Step 1: Extract Inputs

Same as Black-Scholes (S, K, T years, r, q, σ, opt_type) plus:

| Field | Notes | Default |
|---|---|---|
| n_steps | Tree depth | 200 |
| style   | "american" or "european" | american |

For discrete dividends, use the escrowed-dividend trick (subtract PV of dividends from S before building the tree).

---

## Step 2: Build the Tree

```
dt = T / n
u  = exp(σ · √dt)
d  = 1 / u
p  = (exp((r − q)·dt) − d) / (u − d)
disc = exp(−r·dt)
```

If `p ∉ [0, 1]`, the tree is ill-conditioned (e.g., σ too small relative to drift over dt) — **stop and report**. Increase `n_steps` or reduce |r − q|·dt.

---

## Step 3: Terminal Payoff

At node (i, j) with `i = 0..n` and `j` = number of up-moves:
```
S_ij = S · u^j · d^(i − j)
payoff_call_i = max(S_ni − K, 0)
payoff_put_i  = max(K − S_ni, 0)
```

---

## Step 4: Backward Induction

For each time step i from n−1 down to 0, for each node j ≤ i:
```
S_ij = S · u^j · d^(i − j)
continuation = disc · (p · V_(i+1, j+1) + (1 − p) · V_(i+1, j))
if style == "american":
    V_ij = max(continuation, max(S_ij − K, 0))   # call
    V_ij = max(continuation, max(K − S_ij, 0))   # put
else:
    V_ij = continuation
```

`V_00` is the option price.

---

## Step 5: Sanity Checks

1. Re-run with `style = "european"` and compare to your Black-Scholes price → residual must be < 1% for n ≥ 200 (converges as O(1/n)).
2. Early-exercise premium ≥ 0 always. If American < European, your tree has a bug.
3. **Doubling n_steps** should roughly halve the BS-vs-tree residual.

---

## Step 6: When Is Early Exercise Optimal?

- **American call on non-dividend stock**: never optimal — premium = 0 (Merton 1973). If your tree shows premium > 0 with q = 0, the tree is buggy.
- **American call on dividend-paying stock**: optimal *just before* ex-div date if dividend > time value of the remaining call.
- **American put**: often optimal when deep ITM and far from expiry (you can earn r on K immediately).

---

## Step 7: Respond to User

Report:
1. American price, European price, early-exercise premium (absolute and as % of American).
2. The optimal exercise boundary at t=0 if requested (the strike level above/below which immediate exercise dominates continuation).
3. Convergence check note: how the price changed when you doubled `n_steps`.

For pure European pricing, recommend `black-scholes` (faster, exact).
