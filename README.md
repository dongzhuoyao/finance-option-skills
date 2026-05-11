<div align="center">

# 🎯 finance-option-skills

### **The AI options desk in your terminal.**

#### Black-Scholes to Bitcoin. Twenty skills. One install.

[![Version](https://img.shields.io/badge/version-0.2.0-blue.svg)](https://github.com/dongzhuoyao/finance-option-skills/releases)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Plugins](https://img.shields.io/badge/plugins-7-purple.svg)](#-whats-inside)
[![Skills](https://img.shields.io/badge/skills-20-orange.svg)](#-skill-catalog)
[![Claude Code](https://img.shields.io/badge/Claude%20Code-ready-d97757.svg)](https://claude.com/claude-code)
[![Deribit](https://img.shields.io/badge/Deribit-integrated-00a8e8.svg)](https://www.deribit.com)

**Stop building options spreadsheets. Start asking Claude.**

</div>

> [!WARNING]
> Educational only. Options trading involves substantial risk of loss. **Not financial advice.** Always DYOR.

---

## ⚡ 60-Second Install

```bash
npx plugins add dongzhuoyao/finance-option-skills
```

That's it. Restart Claude Code, then talk to it like an options trader:

```
You: "Price me an iron condor on SPX, 5700/5800/5900/6000, 30 DTE at 18% IV."
You: "Show the payoff curve."
You: "What's max pain for BTC this Friday?"
You: "Fit the IV surface for QQQ and find the cheapest contracts."
You: "Compute the net Greeks across my book — 4 positions, mixed tickers."
```

Claude reaches for the right skill automatically. **No spreadsheets. No notebooks. No copy-paste.**

---

## 🧠 Why this exists

LLMs are great at talking about options. They're awful at **doing** options work — until you give them the right tools.

This repo packages **20 production-grade skills** into one Claude Code marketplace covering everything from textbook Black-Scholes to inverse-settled Deribit BTC options. Each skill is:

- ✅ **Single-purpose** — one job, done right
- ✅ **No silent fallbacks** — if data is stale, you hear about it
- ✅ **Math-precise** — formulas, units, edge cases all spelled out
- ✅ **Hand-off aware** — skills chain together (chain → IV → payoff → Greeks → sizing)

Adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills) (MIT), narrowed and deepened for **options/derivatives only** — and extended with full Deribit crypto coverage.

---

## 🏗️ What's Inside

### Seven plugin groups, twenty skills

<table>
<tr>
<td width="33%">

**🧮 Pricing**
- Black-Scholes-Merton
- Greeks (1st & 2nd order)
- Binomial trees (American)

</td>
<td width="33%">

**📐 Strategies**
- Interactive payoff curves
- 13 strategy templates
- Strategy selector by view

</td>
<td width="33%">

**🌊 Volatility**
- SVI / SABR surfaces
- Skew (25d RR, butterfly)
- Term structure

</td>
</tr>
<tr>
<td>

**📡 Data**
- yfinance chains
- Broker screenshots
- CBOE VIX / SKEW

</td>
<td>

**🛡️ Risk**
- Delta hedging schedules
- Kelly sizing
- Portfolio Greeks

</td>
<td>

**₿ Crypto (Deribit)**
- BTC / ETH chains
- Inverse-options math
- DVOL + GEX + max pain

</td>
</tr>
<tr>
<td colspan="3" align="center">

**🛠️ Skill Creator** — Author and grade new options skills with a 14-point quality rubric

</td>
</tr>
</table>

---

## 🎬 See It In Action

### Vanilla equity option

> **You:** *"What does a call debit spread on TSLA 350/360 look like, 30 days out, 45% IV?"*

Claude → invokes `options-payoff` → renders an interactive HTML widget with:
- Expiry payoff curve + theoretical curve
- Sliders for every parameter
- Live max profit / max loss / breakeven cards
- Greeks overlay on demand

### Crypto positioning

> **You:** *"Where's the gamma wall for ETH next Friday on Deribit?"*

Claude → `deribit-options-chain` → `gex-max-pain` → reports:
- Max-pain strike + holder-payoff curve
- Top 3 gamma walls by absolute exposure
- Spot's position relative to the walls
- Caveat about dealer-positioning assumptions

### Vol regime check

> **You:** *"Is BTC vol cheap right now?"*

Claude → `dvol-index` → returns:
- Current DVOL + 1y percentile
- IV-RV spread (vol carry context)
- Regime classification (calm/normal/elevated/stressed/crisis)
- Term-structure shape from the chain

---

## 📚 Skill Catalog

<details open>
<summary><b>🧮 Option Pricing</b> — closed-form and numerical pricing under BSM (3 skills)</summary>

| Skill | Description |
|---|---|
| [black-scholes](plugins/option-pricing/skills/black-scholes/) | European call/put pricing with dividend yield, put-call parity verification |
| [greeks-calculator](plugins/option-pricing/skills/greeks-calculator/) | First and second-order Greeks: Δ Γ Θ ν ρ + vanna, volga, charm, color |
| [binomial-pricing](plugins/option-pricing/skills/binomial-pricing/) | Cox-Ross-Rubinstein for American options and early-exercise premium |

</details>

<details open>
<summary><b>📐 Option Strategies</b> — payoff visualization and strategy selection (2 skills)</summary>

| Skill | Description |
|---|---|
| [options-payoff](plugins/option-strategies/skills/options-payoff/) | Interactive payoff curves — verticals, butterflies, condors, calendars, straddles, ratios, jade lizards, broken wings |
| [strategy-selector](plugins/option-strategies/skills/strategy-selector/) | Map (direction × vol view × risk tolerance) to the right multi-leg structure |

</details>

<details open>
<summary><b>🌊 Option Volatility</b> — surface, skew, term structure (3 skills)</summary>

| Skill | Description |
|---|---|
| [iv-surface](plugins/option-volatility/skills/iv-surface/) | Fit SVI / SABR per expiry, run calendar + butterfly arbitrage tests, surface dislocations |
| [vol-skew](plugins/option-volatility/skills/vol-skew/) | 25-delta risk reversal & butterfly with 1y historical percentile context |
| [vol-term-structure](plugins/option-volatility/skills/vol-term-structure/) | ATM IV across expiries — contango/backwardation, earnings vol bumps |

</details>

<details open>
<summary><b>📡 Option Data Providers</b> — yfinance, CBOE, broker readers (3 skills)</summary>

| Skill | Description |
|---|---|
| [yfinance-options](plugins/option-data-providers/skills/yfinance-options/) | Pull chains, Greeks, historicals; resolves "0DTE / weeklies / monthlies" to real dates |
| [options-chain-reader](plugins/option-data-providers/skills/options-chain-reader/) | Parse IBKR / TastyTrade / ToS / Robinhood / Schwab screenshots into structured legs |
| [cboe-data](plugins/option-data-providers/skills/cboe-data/) | VIX / VVIX / SKEW / VIX9D / VIX3M with regime classification |

</details>

<details open>
<summary><b>🛡️ Option Risk Management</b> — hedge, size, aggregate (3 skills)</summary>

| Skill | Description |
|---|---|
| [delta-hedging](plugins/option-risk-management/skills/delta-hedging/) | Rebalance bands, gamma scalp P&L, realized-vol breakeven, TC drag |
| [position-sizing](plugins/option-risk-management/skills/position-sizing/) | Fixed-max-loss + quarter Kelly + BP cap; refuses to size unbounded-loss legs |
| [portfolio-greeks](plugins/option-risk-management/skills/portfolio-greeks/) | Per-underlying + book-level dollar Greeks; concentration flags; 5 stress tests |

</details>

<details open>
<summary><b>₿ Crypto Options (Deribit)</b> — the part that's different (5 skills)</summary>

| Skill | Description |
|---|---|
| [deribit-data](plugins/option-crypto/skills/deribit-data/) | Public REST wrapper — instruments, ticker, book, index, funding, DVOL history |
| [deribit-options-chain](plugins/option-crypto/skills/deribit-options-chain/) | One-call BTC / ETH chains, coin→USD conversion, ATM strip |
| [inverse-options-pricing](plugins/option-crypto/skills/inverse-options-pricing/) | Coin-settled BSM math + Greek conversions for the part that breaks textbooks |
| [dvol-index](plugins/option-crypto/skills/dvol-index/) | Deribit's "crypto VIX" — regime, percentile, IV-RV spread |
| [gex-max-pain](plugins/option-crypto/skills/gex-max-pain/) | Dealer gamma exposure + max-pain strike (with explicit positioning caveats) |

</details>

<details>
<summary><b>🛠️ Skill Creator</b> — extend the marketplace (1 skill)</summary>

| Skill | Description |
|---|---|
| [skill-creator](plugins/option-skill-creator/skills/skill-creator/) | Scaffold + score new options skills against a 14-point quality rubric |

</details>

---

## 🔥 Why crypto support actually matters

Most "options skills" repos pretend equity Black-Scholes works on Deribit. **It doesn't.**

Deribit BTC/ETH options are **inverse-settled**: premium and payoff are denominated in the coin, not USD. This means:

- **Deltas are different.** A Deribit ATM call has delta ~0.35, not 0.50.
- **Mark prices look weird.** A "0.04 BTC" premium is ~$4,000 at BTC=$100k.
- **Greeks need conversion** before hedging — or you'll be off by a factor of S.

The `inverse-options-pricing` skill ships the full conversion math (View 1: USD numeraire ÷ S; View 2: coin-numeraire BSM). Greeks come in **both coin and USD-equivalent** with explicit labels. Put-call parity is checked in inverse form: `C − P = 1 − K/S` (not `S − Ke^(-rT)`).

That's the kind of detail you don't get from "AI helps you trade options" listicles.

---

## 🧰 Install Individual Plugins

Don't want all seven? Install à la carte:

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-pricing
npx plugins add dongzhuoyao/finance-option-skills --plugin option-strategies
npx plugins add dongzhuoyao/finance-option-skills --plugin option-volatility
npx plugins add dongzhuoyao/finance-option-skills --plugin option-data-providers
npx plugins add dongzhuoyao/finance-option-skills --plugin option-risk-management
npx plugins add dongzhuoyao/finance-option-skills --plugin option-crypto
npx plugins add dongzhuoyao/finance-option-skills --plugin option-skill-creator
```

Or just one skill:

```bash
npx skills add dongzhuoyao/finance-option-skills --skill options-payoff
npx skills add dongzhuoyao/finance-option-skills --skill iv-surface
npx skills add dongzhuoyao/finance-option-skills --skill deribit-options-chain
```

Other Claude Code agents (Codex, Gemini CLI, GitHub Copilot, etc.):

```bash
npx skills add dongzhuoyao/finance-option-skills -a <agent-name>
```

---

## 🤖 Install on Hermes Agent

[Hermes](https://github.com/anpicasso/hermes) ships its own skill registry — it can't read Claude Code's `marketplace.json` directly. Use Hermes's URL-based installer with a shell loop:

```bash
# 1. Clone the repo (so you have the references/ folders locally)
git clone https://github.com/dongzhuoyao/finance-option-skills.git
cd finance-option-skills

# 2. Bulk install all 20 SKILL.md files into the 'options' category.
#    --force is needed because some skills embed !`command` blocks for live
#    data injection — Hermes's security scanner flags those but they're safe.
for sd in plugins/*/skills/*/; do
  url="https://raw.githubusercontent.com/dongzhuoyao/finance-option-skills/main/${sd}SKILL.md"
  hermes skills install "$url" --category options --force --yes
done

# 3. Overlay references/ folders (URL install only fetches SKILL.md).
for sd in plugins/*/skills/*/; do
  skill=$(basename "$sd")
  [ -d "${sd}references" ] && cp -R "${sd}references" "$HOME/.hermes/skills/options/$skill/"
done

# 4. Restart Hermes so active sessions pick up the new skills.
hermes gateway restart
```

**Verify** the install:

```bash
hermes skills list | grep options       # expect 20 rows
ls ~/.hermes/skills/options/             # 20 directories, references/ present in 8
```

**Test it**:

```bash
hermes chat
> Pull the BTC options chain from Deribit and show me the ATM strip
```

Should fire `deribit-options-chain`.

**Uninstall**:

```bash
rm -rf ~/.hermes/skills/options/
hermes skills audit       # de-registers the removed skills from Hermes's manifest
```

> **Why not `hermes plugins install`?** It clones the repo to `~/.hermes/plugins/` but warns "doesn't contain plugin.yaml" and doesn't auto-discover SKILL.md files inside `plugins/<group>/skills/<name>/`. Hermes's plugin format expects a flat `skills/<name>/SKILL.md` layout; this repo uses the Claude Code marketplace layout instead. The skills-install loop above is the working path.

---

## 🧩 How skills chain together

The skills are designed to compose. A typical "real" question pulls 3-5 of them automatically:

```
"Find me a cheap BTC call to express my bullish view"
            │
            ▼
   ┌──────────────────┐
   │  dvol-index      │ ─→ "DVOL at 28th pct, vol is cheap"
   └──────────────────┘
            │
            ▼
   ┌──────────────────┐
   │  deribit-data    │ ─→ "BTC = $98,400"
   └──────────────────┘
            │
            ▼
   ┌──────────────────────────┐
   │  deribit-options-chain   │ ─→ filtered chain, ATM strip
   └──────────────────────────┘
            │
            ▼
   ┌──────────────────────────┐
   │  inverse-options-pricing │ ─→ residual vs Deribit mark
   └──────────────────────────┘
            │
            ▼
   ┌──────────────────┐
   │  strategy-selector│ ─→ "long call vs call debit spread vs short put"
   └──────────────────┘
            │
            ▼
   ┌──────────────────┐
   │  options-payoff   │ ─→ interactive widget
   └──────────────────┘
            │
            ▼
   ┌──────────────────┐
   │  position-sizing  │ ─→ "buy 3 contracts, $X at risk"
   └──────────────────┘
```

You ask one question. Claude orchestrates the chain.

---

## 🎓 Design principles

Every skill follows these rules — call out violations as bugs:

| Rule | What it means |
|---|---|
| **No silent fallbacks** | When data is stale or an API fails, you hear about it. No "I'll just use a default" surprises. |
| **Units always labeled** | Vega per vol point. Theta per calendar day. Coin delta vs USD delta. No ambiguity. |
| **Edge cases enumerated** | T ≤ 0 → intrinsic + warning. σ ≤ 0 → undefined + warning. Not silently coerced. |
| **Math, not vibes** | Every payoff comes with the formula. Every Greek has a derivation in `references/`. |
| **Refuses unbounded-loss sizing** | `position-sizing` won't size a naked call. You get a different structure or no structure. |

---

## 🤝 Contributing

Have an options skill that's missing? PR it.

1. Pick a plugin group (or propose a new one)
2. Use `skill-creator` to scaffold
3. Score against the 14-point rubric (must hit ≥ 10)
4. Open a PR

For the engineering bar, see [CLAUDE.md](CLAUDE.md).

---

## 📜 Credit & License

Structure, plugin marketplace design, and parts of the `options-payoff` SKILL.md are adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills) (MIT). This repo focuses exclusively on **options & derivatives**, and adds the full crypto / Deribit layer.

[**MIT License**](LICENSE) — use it, fork it, ship it.

---

<div align="center">

### **Built for the trader who'd rather ship than spreadsheet.**

⭐ **Star this repo** if it saves you one afternoon of pricing math. ⭐

[Report a bug](https://github.com/dongzhuoyao/finance-option-skills/issues) · [Request a skill](https://github.com/dongzhuoyao/finance-option-skills/issues/new) · [See it on Claude Code](https://claude.com/claude-code)

</div>
