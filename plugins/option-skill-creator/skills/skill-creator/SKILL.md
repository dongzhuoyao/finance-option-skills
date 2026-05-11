---
name: skill-creator
description: >
  Create, evaluate, and improve agent skills specifically for the options/derivatives domain.
  Use this skill when the user asks to "create a new skill", "add an options skill for X",
  "evaluate this SKILL.md", "improve this skill", or wants to extend the finance-option-skills marketplace.
  Triggers: "new skill", "skill template", "skill rubric", "evaluate SKILL.md", "what's missing
  from this skill", "add a strategy / data source / risk metric as a skill".
---

# Options Skill Creator

Author and improve SKILL.md files for the `finance-option-skills` marketplace.

---

## Step 1: Clarify Intent

| User says | What they need |
|---|---|
| "Create a skill for X" | New skill scaffold + SKILL.md + README + references |
| "Evaluate this SKILL.md" | Quality rubric scoring |
| "Improve this skill" | Identify weaknesses + concrete edits |
| "What's missing" | Gap analysis vs the rubric |

If unclear, ask: "Do you want to (a) scaffold a new skill, (b) score an existing one, or (c) improve a specific section?"

---

## Step 2: Choose the Plugin Group

Match the proposed skill to one of:

| Group | What goes here |
|---|---|
| **option-pricing** | Pricing models (closed-form, numerical, Monte Carlo) and Greeks |
| **option-strategies** | Strategy construction, payoff visualization, P&L analysis |
| **option-volatility** | IV analysis, surface fitting, vol forecasting |
| **option-data-providers** | New data sources (broker readers, market data APIs, vol indices) |
| **option-risk-management** | Hedging, sizing, portfolio-level analysis |

If the skill doesn't fit any group, **stop** and ask whether to create a new group — don't dump it into the closest match.

---

## Step 3: Generate the Scaffold

For a new skill `my-new-skill` in group `option-X`:
```
plugins/option-X/skills/my-new-skill/
  SKILL.md       # main file
  README.md      # GitHub-facing
  references/    # optional, for long API tables or derivations
```

---

## Step 4: SKILL.md Template

```markdown
---
name: my-new-skill
description: >
  [One-paragraph description of WHEN to use, with trigger keywords inline.]
  Use this skill when [specific user intent]. Triggers: "X", "Y", "Z".
  [Mention defaults so the skill works with partial input.]
---

# Skill Title

[Two-sentence overview of what the skill produces.]

---

## Step 1: [Extract Inputs]
[Table of fields with required/default values.]

## Step 2: [Compute / Fetch / Match]
[Concrete formula or table mapping intent to action.]

## Step 3: [Sanity Checks]
[Explicit list of failure modes — never silently fallback.]

## Step 4: [Respond to User]
[Format of the output. Hand-off to other skills if relevant.]

---

## Reference Files

- `references/foo.md` — [what's in it]
```

---

## Step 5: Quality Rubric (see `references/rubric.md`)

Score 0–2 on each dimension:

| Dimension | 0 | 1 | 2 |
|---|---|---|---|
| **Trigger clarity** | Description is vague | Has triggers but missing keywords | Includes 5+ trigger phrases, no ambiguity |
| **Defaults** | Skill fails without all inputs | Some defaults | All optional fields have sensible defaults |
| **Failure handling** | Silently coerces bad input | Some checks | All failure modes surface explicitly |
| **Hand-off** | No mention of sibling skills | Mentions one | Clearly recommends downstream skills |
| **Math precision** | No formulas / unclear | Formulas present | Formulas + units + edge cases |
| **Examples** | None | Generic | One worked example with real numbers |
| **No silent fallbacks** | Has `if fail: use default` | Mostly explicit | Every failure path either errors or asks user |

A passing skill scores ≥ 10/14.

---

## Step 6: Common Mistakes (from reviewing skills)

| Mistake | Fix |
|---|---|
| Generic trigger ("for options") | List specific keywords ("butterfly", "calendar", "IV crush") |
| No units on Greeks | Always label "per vol pt", "per calendar day" |
| Embeds 200 lines of code in SKILL.md | Move to `references/` and reference by path |
| "Uses defaults when missing" without listing them | Make the default table explicit |
| Silent fallback ("if API fails, use cached") | Either retry or surface the failure — never substitute |
| Recommends real-time trading on stale data | Always note data delay |

---

## Step 7: After Authoring

1. Run skill-lint to validate frontmatter and description length (< 1024 chars).
2. Update root README.md to list the new skill under its plugin group.
3. Bump version in `plugins/<group>/plugin.json` and `.claude-plugin/marketplace.json`.
4. Test the skill by invoking it through Claude Code with a realistic scenario.

---

## Reference Files

- `references/rubric.md` — Full quality rubric with examples of 0/1/2 scoring per dimension
- `references/template.md` — Copy-paste SKILL.md skeleton

## Credit

Rubric structure adapted from [himself65/finance-skills](https://github.com/himself65/finance-skills) `skill-creator`.
