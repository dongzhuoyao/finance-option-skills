---
name: greeks-calculator
description: >
  Compute first-order and second-order option Greeks under Black-Scholes-Merton.
  Use this skill whenever the user asks about an option's sensitivity to spot, time, vol, or rates.
  Triggers include any mention of: delta, gamma, theta, vega, rho, vanna, volga, charm, color,
  speed, zomma, "how much does this lose per day", "what's the delta of this position",
  "gamma scalp", "vol exposure", or asking for the full Greek profile of a single contract or a multi-leg book.
  Also triggers for "DvegaDtime", "DdeltaDvol", and net-Greek questions across positions.
---

# Option Greeks Calculator

Compute the standard Greek family for European options under BSM.

---

## Step 1: Extract Inputs

Same as black-scholes skill: S, K, T (years), r, q, σ, option type. For multi-leg positions, gather all legs and the signed quantity (`+` long, `−` short, multiplier 100).

---

## Step 2: Compute Greeks (per-share, then scale)

```
d1 = [ln(S/K) + (r − q + σ²/2)·T] / (σ·√T)
d2 = d1 − σ·√T
n(d1) = (1/√(2π)) · exp(−d1²/2)        # standard normal PDF
```

### First-Order

| Greek | Call | Put |
|---|---|---|
| **Delta**  Δ | e^(−qT)·N(d1) | e^(−qT)·(N(d1) − 1) |
| **Vega**   ν | S·e^(−qT)·n(d1)·√T (per 1.00 vol; divide by 100 for per-vol-point) | same as call |
| **Theta**  Θ | −S·e^(−qT)·n(d1)·σ/(2√T) − r·K·e^(−rT)·N(d2) + q·S·e^(−qT)·N(d1) | −S·e^(−qT)·n(d1)·σ/(2√T) + r·K·e^(−rT)·N(−d2) − q·S·e^(−qT)·N(−d1) |
| **Rho**    ρ | K·T·e^(−rT)·N(d2) | −K·T·e^(−rT)·N(−d2) |

### Second-Order

| Greek | Formula |
|---|---|
| **Gamma**  Γ | e^(−qT)·n(d1) / (S·σ·√T) — same for call and put |
| **Vanna** (DδDσ) | −e^(−qT)·n(d1) · d2/σ |
| **Volga** (DνDσ) | ν · d1·d2/σ |
| **Charm** (DδDT) | call: −e^(−qT)·n(d1)·[2(r−q)T − d2·σ·√T]/(2T·σ·√T) + q·e^(−qT)·N(d1); put: similar with sign flips |
| **Color** (DΓDT) | −e^(−qT)·n(d1)/(2·S·T·σ·√T) · [2qT + 1 + (2(r−q)T − d2·σ·√T)·d1/(σ·√T)] |

---

## Step 3: Conventions — Always State Them

Practitioners disagree; state your scaling explicitly:

| Greek | Common scaling | This skill's default |
|---|---|---|
| Delta | per $1 of spot | per $1 of spot |
| Gamma | per $1 (raw) or per $1 of spot per $1 (raw) | **raw** (1.00 = position re-deltas at this rate) |
| Vega | per 1.00 vol (raw) vs per 0.01 ("per vol point") | **per vol point** (raw / 100) |
| Theta | per year vs **per calendar day** | **per calendar day** (raw / 365) |
| Rho | per 1.00 rate vs **per 0.01 (1bp × 100)** | **per 1% rate move** (raw / 100) |

Show the scaled numbers AND label them.

---

## Step 4: Position-Level Aggregation

For a multi-leg book:
```
net_greek = Σ (sign_i · qty_i · multiplier_i · greek_i)
```
- Multiplier = 100 for US equity options.
- Sign: +1 for long, −1 for short.

Use the table from `references/greeks_reference.md` for a worked 4-leg iron condor example.

---

## Step 5: Sanity Checks

1. **Call Δ ∈ [0, e^(−qT)]**, **Put Δ ∈ [−e^(−qT), 0]**. Outside → bug.
2. **Gamma ≥ 0** always for long options. Negative implies short positioning.
3. **Vega and Gamma peak near ATM**, asymmetric for high T or high σ.
4. **Put-Call parity check on Δ**: `Δ_call − Δ_put = e^(−qT)`. If off by > 1e-6, your d1 has drift.

If any check fails, **stop and report** — don't paper over.

---

## Step 6: Respond to User

For a single option: a 2-column table of (greek, value, units, "per X move").

For a book: a per-leg table PLUS a "net" row, then a one-line interpretation: e.g., "Net Δ = +37 → equivalent to long 37 shares of underlying. Net vega = −$420/vol-pt → you LOSE $420 if IV climbs 1 point. Theta = +$58/day → you collect $58/day if nothing moves."

Recommend `delta-hedging` if net Δ is large, or `iv-surface` / `vol-skew` if vega exposure dominates.

---

## Reference Files

- `references/greeks_reference.md` — Worked examples, position aggregation table, "what does X exposure feel like" intuition guide
