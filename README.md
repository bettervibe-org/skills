# skills

Agent skills for AI coding harnesses like Claude Code/Codex/Gemini from [bettervibe](https://bettervibe.org).

## Installation

```bash
npx skills add bettervibe-org/skills --skill SKILLNAME
```

## Available Skills

### better-rust

Guidance for writing idiomatic Rust that works with the borrow checker and type system instead of around them.

```bash
npx skills add bettervibe-org/skills --skill better-rust
```

### review-agents-md

Reviews `AGENTS.md` / `CLAUDE.md` context files against best practices and reports concrete cuts, rewrites, and additions.

```bash
npx skills add bettervibe-org/skills --skill review-agents-md
```

### better-skill-names

Audits, critiques, and proposes names for agent skills — validates hard rules, flags style issues, and surfaces inconsistencies across a skill collection.

```bash
npx skills add bettervibe-org/skills --skill better-skill-names
```

### bootstrap-project

Bootstraps a fresh repo for agentic engineering. Asks three questions (language, project kind, external boundaries) and scaffolds a strict toolchain, lefthook pre-commit hooks, one-command bring-up, and a hand-written `AGENTS.md` — small enough to grow into, never something to delete. Architecture is the user's call, not the scaffold's.

```bash
npx skills add bettervibe-org/skills --skill bootstrap-project
```
