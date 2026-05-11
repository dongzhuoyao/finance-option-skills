# skill-creator

Author, evaluate, and improve agent skills for the options/derivatives domain.

## Triggers

- "Create a new options skill for X"
- "Evaluate this SKILL.md"
- "Improve this skill"
- "What's missing"

## What it does

- Scaffolds new SKILL.md files into the correct plugin group
- Scores existing SKILL.md files against a 7-dimension rubric (0–14 points)
- Flags common mistakes (silent fallbacks, missing units, embedded code blocks)
- Provides a copy-paste template

## Platform

Works on **Claude Code** and **Claude.ai**.

## Setup

```bash
npx plugins add dongzhuoyao/finance-option-skills --plugin option-skill-creator
```

## Reference files

- `references/rubric.md` — Quality rubric with worked examples
- `references/template.md` — SKILL.md skeleton
