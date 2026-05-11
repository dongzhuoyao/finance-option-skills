# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A collection of agent skills focused on **options trading and derivatives**, following the [Agent Skills](https://agentskills.io) open standard. Skills are installable into Claude Code, Claude.ai, and other supported agents (Codex, Gemini CLI, GitHub Copilot, etc.). Structure adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills).

This is a **documentation/reference repository** — no build step, no runtime code, no tests. Each "skill" is a markdown file (`SKILL.md`) with structured frontmatter that the agent reads at runtime. Treat new contributions as documentation edits, not code.

## Common commands

There is no compile/test loop. The full set:

```bash
pnpm bump                   # bump versions across marketplace.json + all plugin.jsons (uses ccbump)
git tag v0.X.Y && git push --tags   # triggers release-skills.yml → zips each skill, publishes a GitHub release
```

**Validate SKILL.md frontmatter locally before commit** (the GitHub Actions linter is strict on these):

```bash
python3 - <<'EOF'
import re, yaml
from pathlib import Path
for path in Path('plugins').rglob('SKILL.md'):
    text = path.read_text()
    m = re.match(r'^---\n(.*?)\n---\n', text, re.DOTALL)
    assert m, f'{path}: no frontmatter'
    front = m.group(1)
    yaml.safe_load(front)                        # must parse as YAML
    assert len(front) <= 1024, f'{path}: frontmatter > 1024 chars'
    # angle-bracket check (linter rejects literal < or > in description content;
    # `>` as YAML folded-string indicator on its own line is OK)
    for i, line in enumerate(front.splitlines(), 1):
        stripped = re.sub(r':\s*[>|][-+]?\s*$', ':', line)
        assert '<' not in stripped and '>' not in stripped, f'{path}:{i+1}: angle bracket: {line}'
print('OK')
EOF
```

The CI linter (`himself65/skill-lint@v2`) enforces: frontmatter present, `description` ≤ 1024 chars, no literal `<`/`>` in description content.

## Repository structure

This repo is two things at once:
1. A **Claude Code plugin marketplace** (`.claude-plugin/marketplace.json` + `plugins/`)
2. An **Agent Skills** repository (the `SKILL.md` files inside `plugins/<group>/skills/`)

Skills are organized into plugin groups by usage area within the options domain.

```
.claude-plugin/
  marketplace.json        # Marketplace definition — lists all 7 plugins
plugins/
  option-pricing/         # Black-Scholes, binomial, Greeks
    plugin.json
    skills/
      <skill-name>/
        SKILL.md
        README.md
        references/
  option-strategies/      # Multi-leg payoffs, strategy selection
    plugin.json
    skills/...
  option-volatility/      # IV surface, skew, term structure
    plugin.json
    skills/...
  option-data-providers/  # yfinance, CBOE, broker chain readers
    plugin.json
    skills/...
  option-risk-management/ # Delta hedging, sizing, portfolio Greeks
    plugin.json
    skills/...
  option-crypto/          # Deribit BTC/ETH inverse options, DVOL, GEX, max pain
    plugin.json
    skills/...
  option-skill-creator/   # Skill authoring for the options domain
    plugin.json
    skills/...
.github/workflows/
  release-skills.yml      # Zips each skill and publishes as GitHub release on tag
  skill-lint.yml          # Lints all SKILL.md files
```

## How skills work

Each skill is a self-contained directory under `plugins/<group>/skills/`. The `SKILL.md` file is what Claude reads at runtime — it tells the model when to activate, what steps to follow, and where to find reference details.

### SKILL.md format

```markdown
---
name: skill-name
description: >
  Multi-line description that doubles as the trigger definition.
  Include specific phrases, keywords, and scenarios that should activate this skill.
---

# Skill Title

Step-by-step instructions organized as ## Step N sections.
Tables, code blocks, and formulas as needed.

## Reference Files

- `references/foo.md` — description
```

**Required frontmatter fields:** `name`, `description`

The `description` field is critical — it controls when the skill activates. Write it as a comprehensive trigger list, not a summary. For options skills, include trigger keywords like: strike, expiry, IV, delta, gamma, theta, vega, ATM/ITM/OTM, call/put, spread, straddle, butterfly, condor, calendar, ratio, covered call, naked put, etc.

### Reference files

Markdown documents in `references/` containing detailed API references, code templates, formulas, or schema docs. The SKILL.md instructions tell the model to read specific reference files when needed, keeping the main instructions concise.

### Skill composition (architectural big picture)

Skills in this repo are designed to **chain**. The final step of most SKILL.md files names downstream skills the agent should hand off to. A typical chain has the shape: **data → analysis → action**. Two real chains shipped here:

- **Equity options trade**: `yfinance-options` → `iv-surface` / `vol-skew` → `strategy-selector` → `options-payoff` → `position-sizing`
- **Crypto options trade**: `dvol-index` → `deribit-data` → `deribit-options-chain` → `inverse-options-pricing` → `options-payoff` → `position-sizing`

When adding or editing a skill, **explicitly name the upstream skill that feeds it and the downstream skill that consumes its output** in the final "Respond to User" step. This is what makes the agent pick the right next skill without the user re-prompting.

Two specific composition rules baked into the existing skills:

1. **Greeks must be labeled with their numeraire and scaling.** `greeks-calculator` returns per-vol-point vega and per-calendar-day theta; `inverse-options-pricing` returns both coin-denominated and USD-equivalent. Skills that consume Greeks (e.g., `delta-hedging`, `portfolio-greeks`) assume those conventions — don't break them.
2. **Equity skills assume vanilla Black-Scholes; crypto skills assume inverse settlement.** `options-payoff` is reusable for both, but it's the caller's responsibility to convert coin premiums to USD before passing them in (`deribit-options-chain` does this via the `usd_mid` column).

## Creating a new skill

1. Choose the appropriate plugin group (`option-pricing`, `option-strategies`, `option-volatility`, `option-data-providers`, `option-risk-management`, or `option-crypto`)
2. Create `plugins/<group>/skills/<skill-name>/` directory
3. Write `SKILL.md` with YAML frontmatter (`name`, `description`) and step-by-step instructions
4. Add reference files under `references/` for detailed API docs, code templates, or formulas that would bloat the main instructions
5. Add a `README.md` for the skill's GitHub page (description, triggers, platform, setup, reference file list)
6. Update the root `README.md` to list the new skill in the appropriate plugin group table
7. Run the local SKILL.md validation snippet (see "Common commands") before commit
8. Bump versions across `marketplace.json` and every `plugin.json` (`pnpm bump`)
9. The skill will be auto-zipped and released on tag push via GitHub Actions

### Platform considerations

Skills that require shell access, network calls, or external binaries (e.g., yfinance, pip install) only work on **CLI-based agents** like Claude Code. They do **not** work on Claude.ai, which runs in a sandboxed environment that restricts network access and binaries.

Skills that only use Claude's built-in tools (e.g., `show_widget` for payoff charts) work on **Claude.ai**.

### Dynamic content with `!`command``

Skills can embed shell commands that Claude Code executes at skill invocation time, injecting the output inline. Use this for runtime environment checks (tool installation status, auth state, live data). Syntax: wrap in a fenced code block with `` !`command` ``.

Example — fetching live VIX for IV regime context:
```
!`python3 -c "import yfinance as yf; print(f'VIX = {yf.Ticker(\"^VIX\").fast_info[\"lastPrice\"]:.2f}')" 2>/dev/null || echo "VIX unavailable"`
```

Guidelines:
- Use for environment/auth checks so the model skips install/auth steps when unnecessary
- Use for injecting live data (e.g., current spot, VIX) to replace hardcoded values
- Keep commands fast (< 2s) — they run synchronously before the skill loads
- Always include fallback output (e.g., `|| echo "UNAVAILABLE"`) so the skill degrades gracefully
- Only works on CLI-based agents (Claude Code) — Claude.ai ignores these

### Instruction style guidelines

- Organize as numbered steps (## Step 1, Step 2, etc.)
- Use tables to map user intents to actions/methods
- Include defaults for missing parameters so the skill works with partial input
- Put lengthy code templates and API references in `references/` files, not inline
- End with a "Respond to the User" step describing how to present results

## Plugin system

This repo ships as a Claude Code plugin marketplace containing 7 plugins:

| Plugin | Description |
|---|---|
| `option-pricing` | Black-Scholes, binomial trees, Greeks |
| `option-strategies` | Multi-leg payoffs and strategy selection |
| `option-volatility` | IV surface, skew, term structure |
| `option-data-providers` | yfinance, CBOE, broker chain readers |
| `option-risk-management` | Delta hedging, sizing, portfolio Greeks |
| `option-crypto` | Deribit BTC/ETH inverse options, DVOL, GEX, max pain |
| `option-skill-creator` | Authoring/evaluating new skills |

- `.claude-plugin/marketplace.json` — marketplace listing with all 7 plugin entries.
- `plugins/<group>/plugin.json` — per-plugin manifest (name, version, keywords). Skills under `plugins/<group>/skills/` with SKILL.md frontmatter are auto-discovered by the plugin loader.

Users install all plugins via `npx plugins add dongzhuoyao/finance-option-skills`. Individual plugins can be installed via `npx plugins add dongzhuoyao/finance-option-skills --plugin <plugin-name>`. Individual skills via `npx skills add dongzhuoyao/finance-option-skills --skill <name>`.

When a skill is invoked as a plugin, it is namespaced as `<plugin-name>:<skill-name>` (e.g., `/option-strategies:options-payoff`).

### Installing on non-Claude-Code agents

The `npx plugins add` / `npx skills add` commands above only work for agents that read the Claude Code marketplace format. Hermes ships its own registry and requires a different install flow — see the **"Install on Hermes Agent"** section in `README.md` (URL-loop install + manual `references/` overlay; `hermes plugins install` does NOT work because the repo lacks `plugin.yaml`).

For new agents not yet documented, the lowest-common-denominator path is: clone the repo, then have the agent read SKILL.md files directly from `plugins/<group>/skills/<name>/`.

## CI/CD

- **Release workflow** (`.github/workflows/release-skills.yml`): On tag push (`v*`), zips each skill from `plugins/*/skills/*/` and publishes them as a GitHub release.
- **Lint workflow** (`.github/workflows/skill-lint.yml`): Lints all `SKILL.md` files across all plugin groups. The linter caps `description` at 1024 chars and rejects angle brackets (`<` / `>`).

## Important constraints

- **No trade execution.** All brokerage-related skills must be read-only. Never allow AI to execute trades or modify orders.
- **No silent fallbacks.** When a prescribed check, tool call, API request, or verification step fails, stop and surface the failure with its specific cause. Do not substitute a weaker check, a default value, or an inferred answer to keep going.
- This is primarily a documentation/reference repository — most of the codebase is `SKILL.md` files with no build step.
- All examples are educational. Real trading requires real risk management beyond what any skill encodes.
