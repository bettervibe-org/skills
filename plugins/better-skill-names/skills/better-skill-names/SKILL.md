---
name: better-skill-names
description: Audits, critiques, and proposes names for agent skills (the `name` field in SKILL.md). Use when the user asks to name a new skill, rename one, audit a skill collection for naming consistency, or says things like "what should I call this skill", "review my skill names", or "is this a good skill name".
---

The meta-principle: a skill name is a discovery handle, not a label. It has to (1) survive the hard rules, (2) match the patterns of its siblings, and (3) tell Claude at a glance what activity it captures. Names that fail any of these waste discovery surface.

## Workflow

Pick the mode from the user's request:

- **Naming a new skill** → run *Generate*.
- **Renaming / critiquing one name** → run *Critique*.
- **Auditing a directory of skills** → run *Audit*.

Always validate against *Hard rules* first — a name that violates them is invalid regardless of style.

## Hard rules (non-negotiable)

A skill `name` MUST:

- Be **≤ 64 characters**.
- Contain **only lowercase letters, numbers, and hyphens** (`a-z`, `0-9`, `-`).
- Not contain reserved words: `anthropic`, `claude` (anywhere in the name).
- Not contain XML tags or whitespace.
- Match the directory name and the `name` frontmatter field (these must agree).

Reject any candidate that fails these. Don't try to "soften" — propose a replacement.

## Style rubric

### Form: prefer gerund, allow consistent alternatives
*Good*: `processing-pdfs`, `analyzing-spreadsheets`, `writing-documentation` — gerund form (verb + -ing) is Anthropic's recommended default.
*Acceptable*: noun phrases (`pdf-processing`) or imperatives (`process-pdfs`) — but only if the rest of the user's collection uses the same form.
*Bad*: a collection that mixes forms (`processing-pdfs` next to `pdf-tool` next to `analyze-data`). Pick one form and migrate.

### Specificity
*Good*: names that name the activity — `reviewing-prs`, `generating-migrations`.
*Bad*: vague labels (`helper`, `utils`, `tools`, `assistant`) or overly generic (`documents`, `data`, `files`). These collide with other skills and tell Claude nothing about when to fire.

### Scope match
*Good*: name covers exactly what the skill does. `commit-message-writing` for a skill that writes commit messages.
*Bad*: name promises more than the skill delivers (`git` for a commit-message-only skill) or less (`pdf-text-extraction` for a skill that also fills forms and merges). Rename to match the actual `description`.

### Sibling consistency
*Good*: `better-rust`, `better-agents-md`, `better-skill-names` — same prefix, same form, easy to scan.
*Bad*: ad-hoc names that ignore the convention already established by neighbors. When auditing, treat the dominant pattern as the rule and flag outliers.

### Reserved-word adjacency
*Good*: names that describe the activity without invoking the model.
*Bad*: `anthropic-helper`, `claude-tools`, `claude-md-fixer` — even if the substring rule didn't forbid it, it's noise. Rename to the activity.

### No version suffixes or status markers
*Good*: stable names — `migrating-databases`.
*Bad*: `migrating-databases-v2`, `migrating-databases-new`, `migrating-databases-final`. Skills evolve in place; version markers rot.

### Pair with the description
The name is half the discovery signal; the `description` is the other half. After settling on a name, sanity-check: does the description say *what it does* AND *when to use it*, in third person? If not, flag the description too — a great name with a vague description still won't trigger.

## Modes

### Generate

Input: a draft `description`, or a one-line summary of what the skill does.

1. Extract the core verb and object (e.g., "review PRs", "extract PDF text").
2. Detect the user's existing convention — read sibling skills if a directory was provided, otherwise default to gerund.
3. Propose **3–5 candidates**, ranked. For each: the name, the form (gerund / noun-phrase / imperative), and a one-line justification.
4. Validate each against *Hard rules*. Drop or fix any that fail.
5. End with a recommendation and the runner-up.

### Critique

Input: a single name (and optionally its description and siblings).

1. Run *Hard rules*. If it fails, stop and propose a fix.
2. Run each *Style rubric* item. For each issue, quote the problem and propose a concrete replacement — don't just label it "vague".
3. End with a verdict: **keep** / **rename to X** / **rename + rewrite description**.

### Audit

Input: a directory of skills (or a list of names).

1. Read every SKILL.md frontmatter; collect `name` + `description`.
2. Run *Hard rules* on each — list violators.
3. Detect the dominant form (gerund / noun / imperative). Flag every name that deviates.
4. Flag vague names, scope mismatches, and reserved-word adjacency.
5. For each flagged name, propose a rename.
6. Verdict: how many names pass, how many to rename, and the dominant form the user should standardize on.

## Output format

For *Critique* and *Audit*, structure the output as:

```
## Hard-rule violations
- `claude-helper`: contains reserved word "claude". Rename to `prompt-helper`.

## Style issues
- `utils`: vague label. Replacement: `formatting-changelogs` (matches sibling form).
- `pdf-tool`: noun-phrase outlier in a gerund collection. Replacement: `processing-pdfs`.

## Description mismatches
- `git`: name promises broad scope; description only covers commit messages. Rename to `writing-commit-messages` OR broaden the skill.

## Verdict
8 names. 1 hard-rule violation, 2 style issues, 1 scope mismatch. Dominant form: gerund — standardize on it.
```

For *Generate*, structure as:

```
## Candidates
1. `reviewing-prs` (gerund) — matches sibling form; names the activity directly.
2. `pr-review` (noun) — shorter; use only if the collection is noun-form.
3. `auditing-pull-requests` (gerund) — explicit but long (22 chars, fine).

## Recommendation
`reviewing-prs`. Runner-up: `auditing-pull-requests` if "review" reads as too generic.
```

Be specific. Always propose a concrete replacement, never a label. If you reject a name, the next line is the fix.
