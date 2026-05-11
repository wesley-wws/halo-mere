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
- **`/retro`** — **Procedural skill** (distinct from the thinking skills): runs a fixed locate → sample → cluster → report workflow over the current session's jsonl transcript and produces a structured "what went wrong / why / how to improve" report inline. Algorithmic details (path encoding, sampling anchors) live in `skills/retro/references/` (loaded on execution, not on trigger). Report only — never modifies files. Defaults to the segment since the last `/clear`; supports `--full`, `--last`, and `<session-id>`.
- **`/architecture-thinking`** — A guiding skill for framework / architecture design. Auto-triggers on review / design / decision phrasings (中英文，e.g. "评审架构", "怎么搭目录", "该不该拆", "should I split this module"); also user-invocable. Three modes: reviewing existing structure (A), guiding new design (B), debating a specific structural decision (C). Built around five principles, four lenses, and the disciplines of a senior architect — not a scoring rubric. Includes anti-over-flagging calibration notes for .NET, Node/TS, and iOS/Swift; other stacks fall back to the universal lenses. Project-specific architecture facts ("we use Clean Architecture with X/Y/Z") are expected to live in the project's `CLAUDE.md`/`AGENTS.md`, not in the skill. **Skill-style** — shapes the reply, does not write files.
- **`/code-decomposition`** — A thinking framework for splitting complex code into focused units. Built on the Single Responsibility question ("what would cause this unit to change?"), applied fractally from a three-line function to a 500-line class to an entire module. Three modes: Review (identify seams) / Execute (six-step workflow) / Debate (single-extraction verdict). Six disciplines (read the whole unit first; refuse to extract clusters you can't name; distinguish complex-single-responsibility from multi-responsibility; reach for the lightest language-native grouping before extracting a new type; match request scope; match user's language). Auto-triggers on refactor / "god class" / "long method" / "extract" / "split" / "拆一下" / "职责太多" phrasing. Stack-specific anti-biases only for JS/TS (barrel files, long Node coordinators); other stacks rely on the universal disciplines. **Skill-style** — shapes the reply, does not write files.
- **`/karpathy-guidelines`** — Behavioral guidelines to reduce common LLM coding mistakes (derived from Karpathy's observations on LLM coding pitfalls). Surface assumptions before coding, prefer surgical changes, define verifiable success criteria. Biases caution over speed; for trivial tasks, use judgment. Auto-triggers when writing / reviewing / refactoring code.
- **`/tidy-knowledge`** — Editorial pass at pause / handoff / pre-commit moments; reconciles `CLAUDE.md` / `AGENTS.md`, `README.md`, and the project's doc surface against the current code. Edits content, surfaces structural recommendations rather than auto-creating sections. Halts if neither agent-facing file exists.

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
