# skills

Agent skills for AI coding harnesses like Claude Code/Codex/Gemini from [unvibe](https://unvibe.org).

## Installation

```bash
npx skills add unvibe-org/skills --skill SKILLNAME
```

## Available Skills

### better-rust

Guidance for writing idiomatic Rust that works with the borrow checker and type system instead of around them.

```bash
npx skills add unvibe-org/skills --skill better-rust
```

### review-agents-md

Reviews `AGENTS.md` / `CLAUDE.md` context files against best practices and reports concrete cuts, rewrites, and additions.

```bash
npx skills add unvibe-org/skills --skill review-agents-md
```

### better-skill-names

Audits, critiques, and proposes names for agent skills — validates hard rules, flags style issues, and surfaces inconsistencies across a skill collection.

```bash
npx skills add unvibe-org/skills --skill better-skill-names
```

### bootstrap-project

Bootstraps a fresh repo for agentic engineering. Asks three questions (language, project kind, external boundaries) and scaffolds a strict toolchain, lefthook pre-commit hooks, one-command bring-up, and a hand-written `AGENTS.md` — small enough to grow into, never something to delete. Architecture is the user's call, not the scaffold's.

```bash
npx skills add unvibe-org/skills --skill bootstrap-project
```

### vibe-check-agentic-engineering

Grades a repository on how well it's set up for agentic engineering — strict types, one-command bring-up, pre-commit hooks, dead-code guardrails, doc/code drift checks, agent-runnable end-to-end tests, parallel review panels, blast-radius friction, and agent-tuned tooling. Produces a Vibe Score 0-100 with verdict.

```bash
npx skills add unvibe-org/skills --skill vibe-check-agentic-engineering
```
