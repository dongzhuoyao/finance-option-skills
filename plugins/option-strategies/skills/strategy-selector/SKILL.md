---
name: strategy-selector
description: >
  Recommend an options strategy given the user's directional view, volatility view, risk tolerance,
  and capital. Use this skill when the user describes a market thesis but does NOT yet know which
  options structure to use. Triggers: "what strategy should I use", "how should I play this", "I think
  X will rally / sell off / stay flat", "I want to bet on a move but cap loss", "I want to collect
  premium", "what's the best way to express this view", or any question where the user gives a thesis
  and asks for the corresponding multi-leg structure. Do not activate if the user already specifies
  the strategy — use `options-payoff` instead.
---

# Options Strategy Selector

Given a user's market view, return a ranked list of candidate strategies with rationale, then hand off to `options-payoff` to visualize.

---

## Step 1: Extract User View

Probe for these dimensions (ask follow-ups for any that are missing):

| Dimension | Values |
|---|---|
| **Direction** | strongly bullish / mildly bullish / neutral / mildly bearish / strongly bearish |
| **Volatility view** | IV will rise / IV will fall / IV stays flat |
| **Time horizon** | days / weeks / months |
| **Capital** | rough $ size of position |
| **Max loss tolerance** | $ cap, or "uncapped OK" |
| **Skill/experience** | beginner / intermediate / advanced |

Refuse to recommend if the user can't articulate at least direction and time horizon — that's not "missing input", it's "no view".

---

## Step 2: Map View to Strategy Grid

| Direction × Vol View | IV will rise | IV will fall | IV flat |
|---|---|---|---|
| **Strongly bullish** | Long call | Call debit spread, Short put | Call debit spread |
| **Mildly bullish** | Call debit spread | Bull put credit spread, Covered call | Bull put credit spread |
| **Neutral** | Long straddle / strangle | Iron condor, Short straddle (advanced) | Iron butterfly, Calendar |
| **Mildly bearish** | Put debit spread | Bear call credit spread | Bear call credit spread |
| **Strongly bearish** | Long put | Put debit spread, Short call | Put debit spread |

**Capped-loss preference** → always pick the debit-spread / credit-spread variant, not the naked leg.

**Uncapped-loss OK + small account** → reject. Position sizing fails on a single bad day.

---

## Step 3: Apply Filters

Cross out candidates that violate the user's constraints:

- **Strict $ cap on loss** → eliminate naked shorts (naked put, naked call, short straddle/strangle, ratio spreads).
- **Beginner**: stick to debit spreads, covered calls, cash-secured puts. No calendars, no ratios, no naked.
- **0DTE preference**: only suggest spreads with daily expiries available on the underlying. SPX/SPY have 0DTE; most names don't.
- **Earnings catalyst within horizon**: prefer long-vol structures (straddle/strangle) since IV crush after earnings will hurt short-vol ones unless you specifically want to SELL the earnings vol pump.

---

## Step 4: Rank Top 3

For each candidate, surface:
- **Why this matches the view** (1 sentence)
- **Approx max profit / max loss** at user's capital
- **Greeks profile** (long/short on each of Δ, Γ, ν, Θ)
- **Best- and worst-case scenarios**

---

## Step 5: Hand Off to options-payoff

Pick the top-1 candidate, propose strikes anchored to current spot, then call `options-payoff` to visualize the payoff curve. If the user wants to see alternatives, render one chart per finalist.

---

## Step 6: Respond to User

Present as a comparison table, then ONE strong recommendation with the rationale and a note on the failure mode (what kills this trade). End with a question: "Want me to show the payoff for [top pick] with strikes set at [proposed K1, K2]?"

---

## What This Skill Won't Do

- Predict direction. The user has to bring the thesis.
- Predict vol. Pair with `vol-term-structure` / `iv-surface` if the user wants a vol-regime read.
- Tell the user whether to take the trade. Final go/no-go is theirs.
