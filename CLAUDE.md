# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A collection of agent skills focused on **options trading and derivatives**, following the [Agent Skills](https://agentskills.io) open standard. Skills are installable into Claude Code, Claude.ai, and other supported agents (Codex, Gemini CLI, GitHub Copilot, etc.). Structure adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills).

## Repository structure

This repo is two things at once:
1. A **Claude Code plugin marketplace** (`.claude-plugin/marketplace.json` + `plugins/`)
2. An **Agent Skills** repository (the `SKILL.md` files inside `plugins/<group>/skills/`)

Skills are organized into plugin groups by usage area within the options domain.

```
.claude-plugin/
  marketplace.json        # Marketplace definition — lists all 6 plugins
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

## Creating a new skill

1. Choose the appropriate plugin group (`option-pricing`, `option-strategies`, `option-volatility`, `option-data-providers`, `option-risk-management`, or `option-crypto`)
2. Create `plugins/<group>/skills/<skill-name>/` directory
3. Write `SKILL.md` with YAML frontmatter (`name`, `description`) and step-by-step instructions
4. Add reference files under `references/` for detailed API docs, code templates, or formulas that would bloat the main instructions
5. Add a `README.md` for the skill's GitHub page (description, triggers, platform, setup, reference file list)
6. Update the root `README.md` to list the new skill in the appropriate plugin group table
7. The skill will be auto-zipped and released on tag push via GitHub Actions

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

- `.claude-plugin/marketplace.json` — marketplace listing with all 6 plugin entries.
- `plugins/<group>/plugin.json` — per-plugin manifest (name, version, keywords). Skills under `plugins/<group>/skills/` with SKILL.md frontmatter are auto-discovered by the plugin loader.

Users install all plugins via `npx plugins add dongzhuoyao/finance-option-skills`. Individual plugins can be installed via `npx plugins add dongzhuoyao/finance-option-skills --plugin <plugin-name>`. Individual skills via `npx skills add dongzhuoyao/finance-option-skills --skill <name>`.

When a skill is invoked as a plugin, it is namespaced as `<plugin-name>:<skill-name>` (e.g., `/option-strategies:options-payoff`).

## CI/CD

- **Release workflow** (`.github/workflows/release-skills.yml`): On tag push (`v*`), zips each skill from `plugins/*/skills/*/` and publishes them as a GitHub release.
- **Lint workflow** (`.github/workflows/skill-lint.yml`): Lints all `SKILL.md` files across all plugin groups. The linter caps `description` at 1024 chars and rejects angle brackets (`<` / `>`).

## Important constraints

- **No trade execution.** All brokerage-related skills must be read-only. Never allow AI to execute trades or modify orders.
- **No silent fallbacks.** When a prescribed check, tool call, API request, or verification step fails, stop and surface the failure with its specific cause. Do not substitute a weaker check, a default value, or an inferred answer to keep going.
- This is primarily a documentation/reference repository — most of the codebase is `SKILL.md` files with no build step.
- All examples are educational. Real trading requires real risk management beyond what any skill encodes.
