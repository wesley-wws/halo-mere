# CLAUDE.md

## Project Overview

halo-mere is a Claude Code plugin providing personal workflow skills, authored by Wesley. No build system, no runtime dependencies — skills are declarative Markdown files with YAML frontmatter.

## Architecture

**Skill-based plugin** following the Claude Code plugin system:
- Each skill lives in `skills/<skill-name>/SKILL.md`
- Skills use YAML frontmatter (`name`, `description`, `argument-hint`) + markdown body
- Plugin metadata in `.claude-plugin/plugin.json` and `.claude-plugin/marketplace.json`
- Workspace permissions in `.claude/settings.json` (denies read access to `archived/*`)
- Archived materials in `archived/` are legacy reference — not active code

## Active Skills

- **`/brainstorm`** — Three-phase guided exploration (extraction → refinement → evaluation).
- **`/structured-output`** — Formatting principles for precise output (8 principles, 5 formatting rules, 6 anti-patterns).

## Development Workflow

No build, lint, or test commands. Development is editing Markdown skill files directly.

- **Install plugin locally**: `/plugin marketplace add wesley-wws/halo-mere` then `/plugin install halo-mere@wesley`
- **Reload after changes**: `/reload-plugins`
- **Version bumps**: `/bump [major|minor|patch]` — updates version in both `plugin.json` and `marketplace.json`

## Conventions

- Skill directories use kebab-case
- One `SKILL.md` per skill directory
- Skills use adaptive conversational flow, not rigid templates
- `.gitignore` excludes `*.local.*` files and session-generated markdown
