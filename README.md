# halo-mere

Personal workflow toolkit for Claude Code — lightweight, flexible.

## Skills

### `/brainstorm`

Guided brainstorming facilitator. Activates when you're exploring a problem, weighing options, or haven't committed to a direction yet.

- Adaptive, conversational exploration across three phases: extraction → refinement → evaluation
- Surfaces trade-offs rather than jumping to solutions
- Resists premature convergence — explicitly asks "is there a perspective we haven't considered yet?" before evaluating

**Trigger examples:** "help me think through X", "I can't decide between A and B", "what are my options for Y"

### `/structured-output`

Formatting principles for precise, well-structured written output. Useful when output spans multiple sections, will be referenced later, or will be consumed by AI downstream.

Covers structural principles (lead with what matters, MECE grouping, logical ordering) and precision principles (terminology consistency, explicit over implicit, specific over general).

### `/retro`

Session retrospective. Reads the current Claude Code session's `.jsonl` transcript and produces a structured "what went wrong / why / how to improve" report — never modifies any file.

A **procedural skill** (as opposed to the thinking skills like `/brainstorm` and `/architecture-thinking`) — it runs a fixed locate → sample → cluster → report workflow. The algorithmic details (path encoding, sampling anchors, segment detection) live in `references/` files loaded on execution, keeping the main skill body lean.

- Default: retros the segment since the last `/clear`; auto-falls back to the previous segment if you just cleared
- Flags: `--full` (entire session), `--last` (previous finished session), `<session-id>` (specific session)
- Auto-triggers on retrospective phrasing in English and Chinese ("retro this session", "what went wrong", "after-action review", "复盘一下", "回顾本次会话", "总结一下问题")

**Trigger examples:** "retro this session", "复盘一下", "what went wrong this round"

### `/architecture-thinking`

A guiding skill for framework / architecture design — a way of thinking, not a checklist. Applies in three modes:

- **Mode A — Reviewing** an existing codebase's structure
- **Mode B — Designing** a new system or module structure from scratch
- **Mode C — Debating** a specific decision (split / merge / module placement / boundary)

Built on five principles (Occam's Razor, Dependency Inversion at the Module Level, Entropy Containment, Consistency as Load-Bearing Structure, Don't Reinvent the Wheel), four lenses (intent / dependency direction / location of complexity / pressure of growth), and the disciplines of a senior architect (read the build graph first; identify the intended style before judging; distinguish architecture from implementation; resist new boundaries).

Includes anti-over-flagging calibration notes for .NET, Node/TypeScript (frontend + backend), and iOS/Swift; other stacks fall back to the universal lenses. Project-specific architecture facts ("we use Clean Architecture with X/Y/Z", "our Common module is intentionally narrow") are expected to live in the project's `CLAUDE.md`/`AGENTS.md` — the skill brings the thinking framework, the project brings the project facts. Always produces a "Do Not Change" section in review mode to resist over-architecture.

Auto-triggers on review / design / decision phrasings in both English and Chinese; also user-invocable. **Skill-style** — shapes how the reply is structured and reasoned, replies inline; does not write files.

### `/code-decomposition`

A thinking framework for breaking complex code into focused, composable units. The aim is not to follow rules mechanically but to find the natural seams in the code and cut along them.

- Built around the Single Responsibility question: **"What would cause this unit to change?"** — applied fractally from a three-line function to a 500-line class to an entire module
- Three modes: **Review** (where could this be decomposed?), **Execute** (decompose this for me), **Debate** (is this specific extraction worth it?)
- Six disciplines that separate careful decomposition from destructive churn — read the whole unit first, refuse to extract clusters you can't name, distinguish complex-single-responsibility from multi-responsibility, reach for the lightest language-native grouping before extracting a new type, match the scope of the user's request, match the user's language
- Stack-specific anti-biases only where the universal principles don't catch the mistake (currently JS/TS: barrel files masquerading as decomposition, long Node coordinators ≠ god functions). Other stacks fall back to the universal disciplines.
- Auto-triggers on refactoring / restructuring / complexity-review requests, in both English and Chinese ("god class", "long method", "extract", "split", "break up", "decompose", "拆一下", "重构", "职责太多", "怎么拆"). **Skill-style** — shapes how the reply is structured and reasoned, replies inline; does not write files.

**Trigger examples:** "this class is doing too much", "help me split this", "把这个函数拆一下", "extract a helper from this"

### `/karpathy-guidelines`

Behavioral guidelines to reduce common LLM coding mistakes, derived from [Andrej Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls.

- Surface assumptions and trade-offs before coding instead of hiding confusion
- Prefer surgical changes over broad rewrites
- Define verifiable success criteria up front
- Bias: caution over speed — for trivial tasks, use judgment

**When it activates:** writing, reviewing, or refactoring code.

### `/tidy-knowledge`

Editorial pass over project documentation at natural pause points (before commit, before merge, end of session, handoff). Reconciles `CLAUDE.md` / `AGENTS.md`, `README.md`, and the project's doc surface against the current code. Halts if neither agent-facing file exists.

- Edits content; surfaces structural changes as recommendations rather than auto-creating sections
- Project scope only — does not touch personal memory or global config

**Trigger examples:** "tidy docs", "整理一下文档", "sync up before commit"

---

## Installation

### Requirements

- [Claude Code](https://claude.ai/claude-code) CLI installed

### Step 1 — Add as a marketplace

In any Claude Code session, run:

```
/plugin marketplace add wesley-wws/halo-mere
```

### Step 2 — Install the plugin

```
/plugin install halo-mere@wesley
```

Or use the interactive UI: run `/plugin`, go to the **Discover** tab, and select `halo-mere`.

### Step 3 — Reload

```
/reload-plugins
```

---

## Usage

Once installed, invoke skills directly in any Claude Code session:

```
/brainstorm how should I structure this API?
/structured-output
/retro
/architecture-thinking
/code-decomposition
/karpathy-guidelines
/tidy-knowledge
```

Skills are automatically suggested by Claude when your request matches their trigger conditions — you don't always need to invoke them explicitly.
