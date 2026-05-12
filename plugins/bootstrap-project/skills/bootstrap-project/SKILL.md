---
name: bootstrap-project
description: Bootstraps a fresh repository for agentic engineering. Asks the user about language and project shape, then scaffolds toolchain pinning, lefthook pre-commit hooks (gitleaks + format + lint), strict compiler/lint settings, one-command bring-up, and a hand-written AGENTS.md. The skill is language-agnostic; the scaffold it produces is language-specific. Use when the user asks to "bootstrap", "scaffold", "init", "set up a project for AI", or "make this repo agent-ready".
---

The meta-principle: ship the smallest setup that makes the agent dangerous on day one. A small project with correct tooling beats a big project with sloppy tooling. Add complexity later when it earns its keep — never on day one. The user can always grow into more; they cannot un-grow a scaffold full of features they didn't ask for.

This skill targets **small-to-medium projects** — single binary, single service, single library. Workspace and monorepo setups are out of scope; if the user asks for one, surface that and check whether this is really day one.

**The skill is language-agnostic; the scaffold it produces is not.** This file describes a fixed set of *needs* every scaffold must satisfy. You translate each need into the best tool for the user's chosen language and write a scaffold that is end-to-end correct for that language. There is deliberately no language matrix here — defaults change, and a hard-coded matrix rots. Use current best practice for the language the user picked, and when two tools are equally reasonable, pick the one with the smaller footprint and the better default config.

## Working assumptions

- The current directory is **blank** (or near-blank: maybe a `.git/`, a `README.md`, a `.gitignore`). Treat any other pre-existing files as user-provided context, not a half-finished scaffold to merge with.
- `git init` if there is no `.git/`. The user expects a repo.
- The user is one human, building one thing. No team conventions to inherit, no CI provider to integrate with, no Docker, no observability, no auth — yet.

## Workflow

1. **Ask the user 3 questions** (see *Questions*). Don't infer — the cost of asking is small, the cost of guessing wrong is a scaffold the user has to delete.
2. **Pick concrete tools** for each *Need* below, given the language and shape the user picked. Pick once, then commit; do not swap tools mid-scaffold.
3. **Generate the scaffold** in this order: toolchain pin → manifest → ignore + secrets files → format/lint config → strict-mode config → lefthook → minimal layout → AGENTS.md → README → one-command bring-up tasks. Ship one trivial passing test so the verify step has something to verify.
4. **Verify**: run the format check, the linter, and the test command. They must all be clean. If any fails, fix it before reporting.
5. **Hand back** with a short "what's next" — the first command to run, and where to write the first piece of code.

## Questions

Ask via `AskUserQuestion`. Three questions, no more:

1. **Language / runtime?** Offer the languages you know well as concrete options plus "Other". For "Other", ask the user to name the closest analogue and any tooling they want to use.
2. **What kind of project?** CLI, service / daemon, library, web app, something else.
3. **External boundaries on day one?** (multi-select) SQL database, HTTP API client, HTTP server, filesystem only, none yet. Skip this question if Q2 was "library" — libraries don't have IO boundaries.

## Needs the scaffold must satisfy

For each need: pick the most appropriate tool for the chosen language *now*, wire it up, and verify it runs.

1. **Toolchain pinning.** The language version and any auxiliary CLIs (lefthook, gitleaks, the linter, the formatter if it isn't bundled) must be pinned in a single file the user can `mise install` from. Use `.mise.toml` by default; reach for an alternative only if the language's ecosystem strongly expects something else.
2. **Manifest with strict mode.** The language's package/build manifest, configured with the strictest reasonable compiler / type-checker settings the language offers — strict by default, not opt-in. The compiler must currently run clean.
3. **Format + lint, both enforced.** A formatter and a linter that:
   - each run as a single command (no per-rule invocation),
   - currently run clean,
   - are wired into the pre-commit hook,
   - are *not* redocumented in AGENTS.md.
   Prefer one tool that does both jobs when the language has one.
4. **Dead-code guardrail.** Unused code, unused imports, and (if the language has separate dependency declarations) unused dependencies must fail the lint step. Lean on what the linter already gives you; reach for a separate tool only if the linter can't do it.
5. **Schema validation at boundaries.** If the project has a non-`none` external boundary, pick a schema/validation library appropriate to that boundary (HTTP/JSON, SQL rows, file formats) and add it as a dependency so the pattern is set on day one. Don't write boundary code yet — just stage the tool. If no boundaries yet, skip — no need to pick a tool the project doesn't use.
6. **Logs to stdout.** A logging library (or the stdlib) wired to print to stdout, with verbosity controllable via env var. Never to a file the agent can't tail.
7. **One trivial passing test.** Use whatever the idiomatic test framework is for the language. The verify step is meaningless without one real test.
8. **Pre-commit feedback loop.** See *Pre-commit feedback loop* below.
9. **One-command bring-up.** See *One-command bring-up* below.
10. **AGENTS.md.** Hand-written, ≤80 lines. See *AGENTS.md* below.

## Layout

Keep the layout minimal. Architecture is the user's call, not the scaffold's. Generate exactly:

```
<src root>/      The idiomatic source directory for the language.
  <entry>.<ext>  One entry point — `main` for binaries, the library entry for libraries.
tests/           One test file with a single passing test.
```

That's it. No subdirectories, no module skeletons, no opinionated split between pure and impure code, no `models/` or `services/` or `utils/`. The user will add structure as the project earns it; pre-imposed structure is something they'd have to delete.

For **libraries**, the entry point is the library root rather than a `main`.

If the language has strong conventions about top-level layout (e.g. Go's `cmd/`, Rust's `src/`, Python's `src/<package>/`), follow them — but still ship just one source file inside.

## Pre-commit feedback loop

Generate `lefthook.yml` in this shape; substitute the concrete commands and globs for the chosen language:

```yaml
pre-commit:
  parallel: true
  commands:
    gitleaks:
      run: mise x -- gitleaks protect --staged --no-banner --redact
    format:
      glob: "<lang glob>"
      run: <format command>
      stage_fixed: true
    lint:
      glob: "<lang glob>"
      run: <lint command>
pre-push:
  commands:
    test:
      run: <test command>
```

Always ship an empty `.gitleaksignore` with a one-line comment ("known-accepted findings go here"). After writing `lefthook.yml`, run `lefthook install` and verify `.git/hooks/pre-commit` exists. If there's no `.git/` yet, `git init` first.

Tests run on `pre-push`, not `pre-commit`, so the commit loop stays fast — but tests still fire automatically before code leaves the laptop.

## One-command bring-up

The user must reach a working state with **one command** after `git clone`. Add two tasks (in `.mise.toml`, `Makefile`, `package.json` scripts, `Justfile` — whatever's idiomatic):

- `setup` — copies `.env.example` → `.env` if missing, installs deps, runs `lefthook install`.
- `check` — format-check + lint + test in one go. The agent will lean on this constantly, and `pre-commit` should call into it.

Document those two commands at the top of the README.

## AGENTS.md (or CLAUDE.md)

Generate a **hand-written** file — never an `/init`-style dump. ≤80 lines on day one. Include only:

- One paragraph: **what this project is**, and what it isn't.
- A **stack one-liner** the agent would otherwise guess wrong (e.g. "Bun 1.x runtime, not Node"; "Postgres + Drizzle, not Prisma"; "Sonnet 4.6 vision via raw HTTP, no SDK").
- **Three to five non-obvious rules** that are NOT enforced by the linter (e.g. "errors carry owned strings, never `&'static`"; "all mutations idempotent — re-running the command is safe"; "no swallowed errors").
- **Pair every "don't" with a "do".** Bare prohibitions make the agent over-explore looking for the allowed path.

Add a `## Layout` or `## Source of truth` section *only* once the project has structure or duplicated state worth pinning down. On day one, neither is true.

Do **not** include: a checklist of what the formatter/linter does, a file-tree dump, generic engineering advice, or anything restated from the manifest. The `review-agents-md` skill exists to delete that content; don't generate it in the first place.

Default to `AGENTS.md`. Use `CLAUDE.md` only if the user explicitly asks for Claude-specific context.

If running inside a Claude Code environment, also create a `CLAUDE.md` symlink pointing to `AGENTS.md` (`ln -s AGENTS.md CLAUDE.md`), so Claude Code's default discovery picks up the same file. Skip if `CLAUDE.md` already exists.

## Verification before handoff

Before reporting "done", run, in order:

1. The format check — must be clean.
2. The linter — must be clean.
3. The test command — must pass. The trivial test from need 7 is what makes this a real signal, not a vacuous pass.
4. `git status` — show the user exactly what was created.

If any step fails, fix it before handing back. A scaffold that doesn't pass its own check is worse than no scaffold.

## Anti-patterns

- **Don't ask 10 questions.** Three, max. If the user wants more options, they'll say so.
- **Don't scaffold features the user didn't ask for** — no auth, no config loader beyond `.env`, no Docker, no CI workflows, no observability, no feature flags. The user can grow into these; they cannot un-grow them.
- **Don't substitute a hard-coded language matrix for judgment.** Pick the right tool for the language *today*, with the smaller footprint when in doubt. If two tools are equally reasonable, the more standard one wins.
- **Don't hand back a scaffold that fails its own check.** Run it.
- **Don't generate an `/init`-style AGENTS.md.** Hand-write five sections from the questions and the layout. No boilerplate.
- **Don't pre-create empty directories.** No `domain/`, `adapters/`, `services/`, `models/`, `utils/`. If a directory has nothing in it, it doesn't exist yet — the user will create the right shape when they hit the problem that needs it.
- **Don't impose an architecture.** No hexagonal core, no clean-architecture split, no DDD layering on day one. Architecture is something the project earns, not something the scaffold imposes.

## Output format

End the run with:

```
## Bootstrapped <project-name>

Language: <language + runtime>
Shape: <CLI | service | library | web app>
External boundaries: <SQL | HTTP-client | HTTP-server | none>

Tools picked:
- toolchain runner: <name>
- format / lint: <name>
- strict mode: <name + flag>
- test runner: <name>
- logs: <library>
- schema validation: <name (or skipped if no boundaries)>

Generated:
- <manifest>
- .mise.toml, .gitignore, .env.example, .gitleaksignore
- <lint/format config>
- lefthook.yml (installed at .git/hooks/pre-commit)
- <src entry file>
- tests/<one trivial passing test>
- AGENTS.md
- README.md

Verified clean: format ✓, lint ✓, test ✓.

Next:
1. Run <the dev command>.
2. Start writing code in <src entry file>. Add structure when the project earns it.
3. Add a line to AGENTS.md the first time the agent guesses wrong — that's the sign of a missing rule.
```

Be specific. The user should be able to `git add . && git commit -m "scaffold"` and have the lefthook hooks fire on the first commit.
