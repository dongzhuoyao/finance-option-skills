# SKILL.md Template

Copy-paste this as a starting point.

```markdown
---
name: SKILL-NAME-IN-LOWERCASE-WITH-HYPHENS
description: >
  One paragraph describing WHEN to use. Be specific about user intents.
  Use this skill when [precise user intent]. Triggers: "kw1", "kw2", "kw3", "kw4", "kw5".
  [Mention defaults so the skill works with partial input.]
---

# Skill Title (Title Case)

[Two-sentence overview of what the skill does and what it produces.]

---

## Step 1: Extract Inputs

| Field | Notes | Default if missing |
|---|---|---|
| S | Spot price (the underlying NOW, not strike) | required |
| K | Strike | required |
| T | Time to expiry in **years** | 30/365 |
| ... | ... | ... |

[Add a live-data injection if relevant:]
```
!`python3 -c "..." 2>/dev/null || echo "DATA_UNAVAILABLE"`
```

---

## Step 2: Compute / Fetch / Match

[Pseudocode, formula, or decision table. Reference `references/...` for long material.]

---

## Step 3: Sanity Checks

1. [Bound check.]
2. [Conservation law check.]
3. [Cross-skill consistency check.]

If any fails: STOP and report. Do not coerce.

---

## Step 4: Respond to User

1. [Headline number with units.]
2. [Decomposition (e.g., intrinsic vs extrinsic).]
3. [Comparison with market (if applicable).]
4. [Hand-off recommendation to a downstream skill.]

---

## Reference Files

- `references/foo.md` — [what's in it, why it's separate from SKILL.md]
```

## Reminders

- Keep frontmatter `description` under 1024 chars (skill-lint will fail otherwise).
- No `<` or `>` in description (the linter rejects them — angle brackets break some YAML parsers).
- Numbered steps make the skill predictable to invoke.
- Cite cross-references by skill name, not file path.
- Include unit labels on every number that has them.
- Put long code (>50 lines) in `references/`, link from SKILL.md.
