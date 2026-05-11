# Skill Quality Rubric

Score each dimension 0–2. Passing skill: total ≥ 10/14.

## 1. Trigger Clarity

| Score | Pattern |
|---|---|
| 0 | "Use this skill for options stuff." |
| 1 | "Use this skill for options pricing. Triggers: pricing, value, BS." |
| 2 | "Use when the user asks for fair value, theoretical price, put-call parity check, BS, or 'is this option overpriced'. Triggers: Black-Scholes, BSM, intrinsic vs extrinsic, dividend yield, risk-free rate." |

## 2. Defaults

| Score | Pattern |
|---|---|
| 0 | "Required: S, K, T, r, σ, q, type." — no defaults, skill fails on partial input |
| 1 | "Required: S, K, type. Other fields default to reasonable values." — defaults unspecified |
| 2 | Explicit default table: `r = 0.043, q = 0, T = 30/365, σ = 0.20 if no quote.` |

## 3. Failure Handling

| Score | Pattern |
|---|---|
| 0 | "If pricing fails, return intrinsic value." (silent fallback) |
| 1 | "If pricing fails, return intrinsic and log a warning." (partial — at least logs) |
| 2 | "If pricing fails: report which input caused it (T ≤ 0, σ ≤ 0, S ≤ 0) and STOP. Do not coerce." |

## 4. Hand-Off

| Score | Pattern |
|---|---|
| 0 | No mention of other skills |
| 1 | Mentions one downstream skill |
| 2 | "For Greeks see `greeks-calculator`. For surface fit see `iv-surface`. For payoff viz see `options-payoff`." |

## 5. Math Precision

| Score | Pattern |
|---|---|
| 0 | "Compute the price." |
| 1 | "Price = S·N(d1) − K·e^(−rT)·N(d2)" without unit labels |
| 2 | Full formulas with d1/d2 definitions, sigma in **annual decimal**, T in **years**, units labeled. Edge cases for T ≤ 0 and σ ≤ 0 spelled out. |

## 6. Examples

| Score | Pattern |
|---|---|
| 0 | None |
| 1 | "Example: S = 100, K = 100, T = 0.25 → price ≈ 4" |
| 2 | A worked iron-condor example with concrete strikes, IVs, theoretical price, and one-line interpretation. |

## 7. No Silent Fallbacks

| Score | Pattern |
|---|---|
| 0 | "If API returns error, use cached value." |
| 1 | "If API errors, retry once with backoff." (still ambiguous on terminal failure) |
| 2 | "If API errors after retry, STOP and surface the error with the request that failed. Do not substitute." |

## Sample Scoring

`black-scholes` skill (this repo):
- Triggers: 2 (lists BSM, theoretical price, put-call parity, intrinsic vs extrinsic, dividend yield)
- Defaults: 2 (explicit table with r = 4.3%, q = 0, T = 30/365)
- Failure: 2 (T ≤ 0 → intrinsic + WARN; σ ≤ 0 → intrinsic + WARN; S, K ≤ 0 → reject)
- Hand-off: 2 (greeks-calculator, iv-surface)
- Math: 2 (full d1/d2, dividend handling, bounds check)
- Examples: 1 (no end-to-end worked one)
- No-silent: 2 (parity check must be ≤ 1e-6 — explicit fail)

Total: 13/14. Pass. Improvement: add a worked example.
