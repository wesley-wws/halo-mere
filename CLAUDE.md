# CLAUDE.md

## Project Overview

halo-mere is a Claude Code plugin providing personal workflow skills, authored by Wesley. No build system, no runtime dependencies — skills are declarative Markdown files with YAML frontmatter.

## Architecture

**Skill-based plugin** following the Claude Code plugin system:
- Each skill lives in `skills/<skill-name>/SKILL.md`
- Skills use YAML frontmatter (`name`, `description`, `argument-hint`) + markdown body
- Packaged as **four overlapping bundles** defined entirely in `.claude-plugin/marketplace.json`: `halo-code`, `halo-think`, `halo-flow`, and the all-in-one `halo-mere`. Each entry uses `source: "./"`, `strict: false`, and an explicit `skills` array selecting a subset of the one flat `skills/` directory. There is deliberately **no `.claude-plugin/plugin.json`** (see Conventions)
- Workspace permissions in `.claude/settings.json` (denies read access to `archived/*`)
- Archived materials in `archived/` are legacy reference — not active code

## Active Skills

- **`/brainstorm`** — Three-phase guided exploration (extraction → refinement → evaluation).
- **`/structured-output`** — Formatting principles for precise output (8 principles, 5 formatting rules, 6 anti-patterns).
- **`/retro`** — **Procedural skill** (distinct from the thinking skills): runs a fixed locate → sample → cluster → report workflow over the current session's jsonl transcript and produces a structured "what went wrong / why / how to improve" report inline. Algorithmic details (path encoding, sampling anchors) live in `skills/retro/references/` (loaded on execution, not on trigger). Report only — never modifies files. Defaults to the segment since the last `/clear`; supports `--full`, `--last`, and `<session-id>`.
- **`/architecture-thinking`** — A guiding skill for framework / architecture design. Auto-triggers on review / design / decision phrasings (中英文，e.g. "评审架构", "怎么搭目录", "该不该拆", "should I split this module"); also user-invocable. Three modes: reviewing existing structure (A), guiding new design (B), debating a specific structural decision (C). Built around five principles, four lenses, and the disciplines of a senior architect — not a scoring rubric. Per-stack anti-over-flagging calibration (.NET, JS/TS, iOS/Swift) and the three mode templates live in `skills/architecture-thinking/references/`, loaded on execution rather than on trigger: a marker table in the body routes to the one calibration file that applies. Other stacks fall back to the universal lenses. Project-specific architecture facts ("we use Clean Architecture with X/Y/Z") are expected to live in the project's `CLAUDE.md`/`AGENTS.md`, not in the skill. **Skill-style** — shapes the reply, does not write files.
- **`/code-decomposition`** — A thinking framework for splitting complex code into focused units. Built on the Single Responsibility question ("what would cause this unit to change?"), applied fractally from a three-line function to a 500-line class to a whole file (above that level the question belongs to `/architecture-thinking`). Three modes: Review (identify seams) / Execute (six-step workflow) / Debate (single-extraction verdict). Six disciplines (read the whole unit first; refuse to extract clusters you can't name; distinguish complex-single-responsibility from multi-responsibility; reach for the lightest language-native grouping before extracting a new type; match request scope; match user's language). Auto-triggers on refactor / "god class" / "long method" / "extract" / "split" / "拆一下" / "职责太多" phrasing. Stack-specific anti-biases only for JS/TS (barrel files, long Node coordinators); other stacks rely on the universal disciplines. **Skill-style** — shapes the reply, does not write files.
- **`/karpathy-guidelines`** — Behavioral guidelines to reduce common LLM coding mistakes (derived from Karpathy's observations on LLM coding pitfalls). Surface assumptions before coding, prefer surgical changes, define verifiable success criteria. Biases caution over speed; for trivial tasks, use judgment. Auto-triggers when writing / reviewing / refactoring code.
- **`/tidy-knowledge`** — Editorial pass at pause / handoff / pre-commit moments; reconciles `CLAUDE.md` / `AGENTS.md`, `README.md`, and the project's doc surface against the current code. Edits content, surfaces structural recommendations rather than auto-creating sections. Halts if neither agent-facing file exists.

## Development Workflow

No build, lint, or test commands. Development is editing Markdown skill files directly.

- **Install locally**: `/plugin marketplace add wesley-wws/halo-mere` then `/plugin install halo-code@wesley` (or `halo-think` / `halo-flow` / the all-in-one `halo-mere`)
- **Reload after changes**: `/reload-plugins`
- **Version bumps**: `/bump [major|minor|patch]` updates `metadata.version` plus all four `plugins[].version` fields in `marketplace.json`. All bundles share one version and release together
- **Verify a packaging change**: `claude plugin validate .` only checks manifest syntax and will pass on a config that fails to load. Real verification is `claude plugin marketplace add <local copy>` + `claude plugin install <bundle>@<mkt>` + `claude plugin list` (status must read `enabled`) + `claude plugin details <bundle>@<mkt>` (skill count must match the `skills` array)

## Conventions

- Skill directories use kebab-case
- One `SKILL.md` per skill directory
- Skills use adaptive conversational flow, not rigid templates
- **Descriptions carry the full trigger surface.** The `description` field is the only part always in context, so every trigger phrase (English *and* Chinese) belongs there, not in the body. Body-only trigger examples never fire.
- **Frontmatter house standard.** `name` + `description` on every skill. `argument-hint` when the skill accepts arguments (`brainstorm`, `retro`). `allowed-tools` **only** where a capability boundary is worth enforcing rather than merely stated: `retro` (read-only, so `Write`/`Edit` are omitted on purpose) and `architecture-thinking`. Leave it off elsewhere; `allowed-tools` genuinely restricts, so an incomplete list breaks the skill. Do **not** write `user-invocable: true` (both user and model invocation are the default; the field is only useful as `false`). `license` appears on `karpathy-guidelines` alone because its content derives from a third-party source and needs attribution.
- **Overlapping skills declare a boundary, but only name a sibling that ships in the same bundle.** The skill that *owns* a territory claims it concretely. The skill that does *not* own it disclaims it by describing the territory, never by naming the owner: when the owner is loaded its own description already claims the territory, so the name is redundant; when it is not loaded, the name is a dangling pointer that routes the user to a skill they do not have. Naming is therefore allowed only where co-installation is guaranteed (`architecture-thinking` ↔ `code-decomposition`, both in `halo-code`). Cross-bundle deferrals stay abstract: `brainstorm` yields to "whichever loaded skill owns that territory" and falls back to answering directly; `tidy-knowledge` excludes code-level cleanup the same way. Territory map: `/brainstorm` (the method of exploring an undecided question) → `/architecture-thinking` (structure above the type level: projects, packages, folders, dependency graph) → `/code-decomposition` (inside the code: functions, types, files).
- **No root `plugin.json`.** Bundle identity lives in the marketplace entries. Adding `.claude-plugin/plugin.json` back makes every bundle fail to load with `conflicting manifests: both plugin.json and marketplace entry specify components`, because the root manifest auto-discovers all of `skills/` while each entry declares its own subset. Verified against the same layout Anthropic's `anthropic-agent-skills` marketplace uses.
- **Bundles are cut by what the skill acts on**, not by topic: `halo-code` (code), `halo-think` (exploration and written output), `halo-flow` (pause/handoff moments). A skill may appear in more than one `skills` array at zero cost, since all entries read the same directory.
- **Moving a skill between bundles is a description change, not just a manifest change.** A concrete `use <name> instead` clause is only valid while both skills share a bundle; if a move breaks that, rewrite the clause to the abstract form before editing `marketplace.json`.
- `.gitignore` excludes `*.local.*` files and session-generated markdown
