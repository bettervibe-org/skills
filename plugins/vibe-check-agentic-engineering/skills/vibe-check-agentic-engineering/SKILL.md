---
name: vibe-check-agentic-engineering
description: Grades a repository on how well it's set up for agentic engineering — strict types, one-command bring-up, pre-commit hooks, dead-code guardrails, doc/code drift checks, agent-runnable end-to-end tests, parallel review panels, blast-radius friction, and agent-tuned tooling. Produces a Vibe Score 0-100 with verdict. Use when Dominik says "vibe check this repo", "run the agentic engineering eval", "score this project", "is this repo agent-ready", or is judging the bettervibe workshop "Best Agentic Engineering" award.
---

# Best Agentic Engineering Vibe Check 🌴

You are running a **Vibe Check** on this repository against the bettervibe agentic engineering checklist. The participant claims they have set up a project that is ready for agentic engineering. The project may be in **any language** (TypeScript, Rust, Go, Python, etc.). Your job is to **verify, not trust** — and tell them whether they're vibing or one merge away from a Vibe Coding Hangover.

## Rules of engagement
- First, identify the stack: read the manifest file (`package.json`, `Cargo.toml`, `go.mod`, `pyproject.toml`, etc.) and note the language, package manager, and toolchain. Every check below is evaluated *for that stack*.
- Inspect the repo directly: read files, run commands, check configs. Do not ask the participant questions you can answer yourself.
- For each item, assign one vibe:
  - 🚀 **Ship-Ready** — solid, agent-ready
  - 👍 **Solid** — works, minor polish needed
  - 🩹 **Patchy** — partial, will hurt later
  - 💀 **Broken** — missing, fix it
- For 🩹 and 💀, quote the exact file/line that's wrong or missing, and give a one-line fix in the participant's chosen stack.
- Do not make changes. This is a vibe check, not a refactor.
- **Narrate as you go.** Before each item, print a one-line status to the user in this exact shape:
  `[n/14] <item name>… <vibe emoji> <one-line evidence>`
  Example: `[3/14] Strict linter… 👍 Solid — eslint config exists but no --max-warnings 0`
  This keeps the run from feeling like a silent audit. Do not batch — print each line as soon as that item is decided.

## Scoring
Each item is worth points based on its vibe:
- 🚀 Ship-Ready = 10 · 👍 Solid = 7 · 🩹 Patchy = 3 · 💀 Broken = 0 · ➖ N/A skipped from total
Compute **Vibe Score = (sum / max possible) × 100**.

Verdict line based on score:
- 90–100: *"Ready to ship. The agent is lucky to work with you."*
- 75–89: *"Solid foundation. Stretch before the next sprint."*
- 50–74: *"Works for now. Drink some water before the agent gets ambitious."*
- 25–49: *"You're going to feel this on Monday."*
- 0–24: *"Cancel your weekend plans. We have work to do."*

## Checklist

### 1. `AGENTS.md` or `CLAUDE.md` exists and is useful
Review this file via a subagent using the **`review-agents-md`** skill from https://github.com/bettervibe-org/skills.
- If the skill is available in your environment, invoke it on the participant's `AGENTS.md` / `CLAUDE.md`.
- If it is **not** available, do **not** install it. Instead, fetch the skill's instructions from the repo above (e.g. `https://raw.githubusercontent.com/bettervibe-org/skills/main/review-agents-md/SKILL.md` or the equivalent path) and load them into your context manually, then apply them as if the skill were running.

Use the skill's findings as your evidence for this item.

### 2. Strictest reasonable type / compiler settings for the stack
*Agents absorb the friction; the codebase keeps the safety.*
- **TypeScript**: `tsconfig.json` has `"strict": true`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`.
- **Rust**: `clippy` at `warn`/`deny` for `clippy::pedantic` or curated strict set; `#![deny(warnings)]` or equivalent in CI.
- **Go**: `staticcheck` / `go vet` wired up; `golangci-lint` with a strict preset.
- **Python**: `mypy --strict` or `pyright` strict; `ruff` with a wide rule selection.
- **Other**: equivalent strict mode for the toolchain.

Compiler/typechecker must currently run clean.

### 3. Strict linter / formatter config
- Linter configured with a strict ruleset, not the default starter.
- Formatter is wired up and enforced (`prettier`, `rustfmt`, `gofmt`, `black`/`ruff format`, etc.).
- Lint and format checks run clean on the current tree.

### 4. Schema validation at boundaries (if applicable)
- API surface uses schema-first tooling: OpenAPI, Protobuf, GraphQL, Zod, Pydantic, `serde` with strict types, etc.
- If no external API surface yet, mark ➖ N/A with a note.

### 5. Business logic separated from I/O
- At least one module of pure logic exists that could be unit-tested without mocking the network/DB/filesystem.
- If everything is glued to I/O, flag it.

### 6. One-command bring-up, uniform across the repo
- Fresh clone reaches a working state with one command. Verify by reading `README.md`, the manifest's script section, `Makefile`, `Justfile`, `mise.toml`, `devenv.nix`, `Taskfile.yml`.
- Single `dev`, single `test`, single `check` (typecheck/compile + lint + test) — regardless of underlying tool.
- In a monorepo / multi-package layout, the **same verbs work the same way in every subdirectory** — `test` in package A and package Z run that package's tests with no per-folder special-casing the agent has to discover.

### 7. Pre-commit feedback loop
- `lefthook.yml`, `.husky/`, `.pre-commit-config.yaml`, `cargo-husky`, or equivalent exists.
- Hooks include: secret scanning (gitleaks or similar), formatter, linter, typecheck/compile, tests on changed files.
- Confirm hooks actually fire — config files alone don't prove it. Inspect `.git/hooks/` or run an empty commit.

### 8. Dead-code guardrail
- Dead-code detector wired into CI or the `check` script. Examples:
  - TypeScript: `knip`, `ts-prune`
  - Rust: `cargo udeps`, `#![deny(dead_code)]`
  - Go: `staticcheck` (`U1000`), `deadcode`
  - Python: `vulture`, `ruff` (F401, F841)
- Currently passes.

### 9. Logs reachable from the terminal
- Dev logs print to stdout or a tailable file — not buried in a UI the agent can't read.
- Bonus 🚀 if a `logs` or `tail` script exists.

### 10. Docs stay in sync with code
*Agents will happily change code and forget the docs. The repo should remind them — or make drift impossible.*
- If docs exist (`README.md`, `/docs`, API reference, etc.), there is a mechanism that flags code-only changes: a pre-commit / lefthook check that warns when source files change without a matching docs touch, a CI rule, or a hook in `AGENTS.md` / `CLAUDE.md`.
- Stronger 🚀: docs are generated or verified from code so they cannot drift — typed API schemas rendered into reference docs, doctest / `--snapshot` checks, `cargo doc` warnings as errors, `rustdoc` examples compiled, generated CLI `--help` snapshots, etc.
- ➖ N/A only if the repo has no user-facing or developer-facing docs at all.

### 11. Agent can self-test its changes end-to-end
*Passing unit tests don't prove the thing works — the server might not even start. Can the agent actually exercise its own changes before declaring done?*
- The repo provides a way for the agent to drive the app end-to-end, appropriate to the stack: a CLI entrypoint / `python -c` / scratch-file harness for libraries, `curl` against a one-command dev server for JSON APIs, Playwright or `agent-browser` / Rodney / equivalent for web UIs.
- The path is discoverable from `AGENTS.md` / `CLAUDE.md` or an obvious script — an agent landing fresh can find it without guessing.
- Output is something the agent can actually read: stdout, captured response bodies, screenshots the agent can see. Not buried in a GUI or human-only dashboard.
- 💀 if the only feedback loop is `npm test` and there is no way for the agent to verify the change works in the real running system.

### 12. Agentic review panel is set up
*With agents writing most of the diffs, a single human reviewer is the bottleneck. The repo should be ready to run a panel of specialist reviewers — locally, before the PR opens.*
- A review entrypoint exists: a slash command (e.g. `/review`), a script, a `Justfile` / `Makefile` target, a CI workflow, or documented instructions in `AGENTS.md` / `CLAUDE.md` that spawn **multiple specialist reviewers in parallel** (best-practices, DRY, `<language>`, security, maintainability, compliance, etc.) — not one generalist pass.
- A `REVIEW.md` (or equivalent) exists that tells reviewers **what *not* to flag** — theoretical risks, unchanged code, speculative "consider library X" suggestions. Without an ignore list the panel becomes noise.
- Review is **tiered** by diff size / risk surface (trivial / lite / full), so frontier-model tokens aren't spent on typo fixes and security-touching diffs don't get a lite pass.
- Bonus 🚀 if the panel is wired to run **before the PR opens** (pre-push hook, local script, agent instruction) and not only as a post-PR CI gate.
- 💀 if there is no agentic review setup at all and the human is the first reviewer of every diff.

### 13. Friction proportional to blast radius
*The agent (or a tired human) should not be able to silently change a file that breaks every host / every consumer / every tenant. Real correctness should be enforced by assertions; this item is about the **speed bump** layered on top.*
- High-blast-radius surfaces are identified somewhere the agent will read: migrations, auth/crypto, infra modules loaded by every host, public API contracts, schema files, deploy/rollback scripts.
- Touching them triggers extra friction beyond the normal pre-commit: a pre-push hook, a `CODEOWNERS`-required review, a "danger zone" check, or a printed checklist of what must have been done first (dry-run deploy, downstream grep, contract assertion added, etc.).
- An **explicit named bypass** exists (e.g. `FOO_DANGER_OK=1`) so the friction doesn't become theatre once the work is done.
- Bonus 🚀 if the friction message links to the **post-mortem or incident** that established the rule — gives the agent a place to read *why*, not just *what*.

### 14. Tooling tuned for the agent
*Failing tools train behaviour. Noisy tools train the agent to ignore the channel. Silent tools leave it stuck.*
- **Clean signal:** known-false-positives are accept-listed (e.g. `.gitleaksignore`, lint suppressions with a reason), so when a hook or check fires, the agent knows it must act — not "probably another spurious one".
- **Actionable failure messages:** when a hook, check, or CI step fails, the output includes the **exact command to fix it** (e.g. "run `git commit --amend -S` to sign", "run `mise run fmt`", "regenerate with `cargo run -p schema`"). Copy-pasteable, not "see CONTRIBUTING.md".
- Spot-check by reading a couple of hook/CI failure paths in the config — do they print remediation, or just exit non-zero?

### 🎁 Bonus: Surprise us
*What did we miss?*
- Call out up to 3 setups in this repo that materially help an agent and **aren't** covered by items 1–14. Examples of the kind of thing that counts: a custom slash command, a well-scoped MCP server, a `justfile` recipe that bundles the whole "fix → check → test" loop, structured failure logs, a scratchpad convention, evals as code, deterministic seeds, etc.
- For each: one line on **what it is** and one line on **why it helps the agent**.
- Not scored. Awarded as 🎁 in the report card. Repos with two or more genuine 🎁s get a *"Vibe Pioneer"* note appended to the verdict.

## Output format

After the live commentary, print a final report that **leads with a TL;DR** so the participant gets the answer before the receipts.

**Render the Report Card as a Unicode box-drawing table** (not a markdown `| ... |` table). Use `┌┬┐├┼┤└┴┘─│` glyphs, with a divider row between every item row — like the example below. Column widths should grow to fit the longest cell. This terminal-native look is part of the bettervibe brand; do not substitute markdown pipes.

Final report template:

```
## 🪩 TL;DR
- **Score:** XX / 100 — <verdict one-liner>
- **Biggest win:** <item name> — <one line>
- **Biggest miss:** <item name> — <one line>
- **Do this now:** <single concrete fix, copy-pasteable if possible>
- **Earned bonuses:** <count, or "none">

## 🌴 Stack detected
Language: ...
Package manager: ...
Toolchain notes: ...

## Vibe Check Report Card

┌─────┬─────────────────────────────────┬──────┬─────────────────────────────────────────────────────────┐
│  #  │              Item               │ Vibe │                        Evidence                         │
├─────┼─────────────────────────────────┼──────┼─────────────────────────────────────────────────────────┤
│ 1   │ AGENTS.md / CLAUDE.md           │ 💀   │ neither file exists                                     │
├─────┼─────────────────────────────────┼──────┼─────────────────────────────────────────────────────────┤
│ ... │ ...                             │ ...  │ ...                                                     │
└─────┴─────────────────────────────────┴──────┴─────────────────────────────────────────────────────────┘

## 🎁 Bonus finds
- ...

## 🎯 Vibe Score: XX / 100

## 💊 Top 3 hangover preventions
1. ...
2. ...
3. ...

## 🪩 Verdict
[one-liner from the score bank]
```

## HTML report artifact

After printing the markdown report, also generate a **single self-contained HTML report** so the participant has a shareable artifact:

1. Read `templates/report.html` from this skill's folder.
2. Fill in the placeholders (see below) and write the result to `vibe-check-report.html` at the root of the repo being checked.
3. Open it in the default browser: run `open vibe-check-report.html` (macOS), `xdg-open vibe-check-report.html` (Linux), or `start vibe-check-report.html` (Windows). Also print the absolute path so the user has it as a fallback.

### Placeholders

| Placeholder | What goes here |
|---|---|
| `{{REPO_NAME}}` | The repo's name (folder name or from `package.json` / `Cargo.toml`). |
| `{{DATE}}` | Today's date, `YYYY-MM-DD`. |
| `{{STACK_LANGUAGE}}` | e.g. `TypeScript`, `Rust`, `Go`, `Python`. |
| `{{STACK_TOOLCHAIN}}` | Package manager and notable tools, separated by ` · ` (middle-dot with spaces). No `+`, no commas. Example: `pnpm · vitest · biome`. |
| `{{SCORE}}` | The numeric vibe score, 0–100, no `/100` suffix (used in both the gauge and CSS). |
| `{{VERDICT}}` | The one-liner from the score bank. |
| `{{TOP_WIN}}` | Same as the TL;DR. |
| `{{TOP_MISS}}` | Same as the TL;DR. |
| `{{DO_NOW}}` | Same as the TL;DR. |
| `{{BONUS_COUNT}}` | e.g. `2 earned 🎁🎁` or `none`. |
| `{{PIONEER_STICKER}}` | If bonuses ≥ 2, insert `<div class="pioneer">Vibe Pioneer</div>`. Otherwise empty string. |
| `{{CATEGORY_CARDS}}` | One `<div class="cat">` per category (see below). |
| `{{ITEM_CARDS}}` | One `<details class="item">` per item 1–14 (see template comments for shape). |
| `{{BONUS_LIST}}` | A `<ul>` of bonus finds, or `<p class="caption">None spotted.</p>` if none. |

### Categories (for the badge wall)

Group items into four categories. Sub-score = sum of points earned / max possible for that category, formatted as `N / max`. Award the badge if the category's sub-score is ≥ 70% of max, otherwise mark it `locked`.

| Category | Items | Badge (earned) |
|---|---|---|
| 🧱 Foundations | 2, 3, 4, 5 | 🛡️ Type-Safe Citizen |
| ⚡ Feedback Loops | 6, 7, 8, 9, 14 | 🚦 Loop Closer |
| 🤖 Agent Enablement | 1, 10, 11, 12 | 🔍 Agent-Ready |
| 🚨 Blast-Radius Safety | 13 | 🛟 Blast-Radius Aware |

### Item card rules

- `data-vibe` attribute: `great` (🚀), `ok` (👍), `warn` (🩹), `bad` (💀), `na` (➖).
- Open by default (`<details open>`) for `warn` and `bad`. Closed for the rest — they're receipts, not action items.
- Body must include the **evidence** (file path / quoted line / reason). For `warn` and `bad`, also include a `<div class="fix"><b>Fix:</b> …</div>` block with the one-line fix in the participant's stack.
- For `na`, body is a one-line reason and no fix block.

### Output discipline

- Single file, no external assets except the Google Fonts link already in the template.
- Don't include the markdown report inside the HTML — the HTML stands alone.
- Don't invent findings to fill the page. If a section is empty, say so visibly (e.g. "None spotted.").
