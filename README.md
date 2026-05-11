# halo-mere

Personal workflow toolkit for Claude Code — lightweight, flexible.

## Skills

### `/brainstorm`

Guided brainstorming facilitator. Activates when you're exploring a problem, weighing options, or haven't committed to a direction yet.

- Adaptive, conversational exploration across three phases: extraction → refinement → evaluation
- Surfaces trade-offs rather than jumping to solutions
- Saves session outcomes to `brainstorm-<topic>.md` for later continuation

**Trigger examples:** "help me think through X", "I can't decide between A and B", "what are my options for Y"

### `/structured-output`

Formatting principles for precise, well-structured written output. Useful when output spans multiple sections, will be referenced later, or will be consumed by AI downstream.

Covers structural principles (lead with what matters, MECE grouping, logical ordering) and precision principles (terminology consistency, explicit over implicit, specific over general).

### `/retro`

Session retrospective. Reads the current Claude Code session's `.jsonl` transcript and produces a structured "what went wrong / why / how to improve" report — never modifies any file.

- Default: retros the segment since the last `/clear`; auto-falls back to the previous segment if you just cleared
- Flags: `--full` (entire session), `--last` (previous finished session), `<session-id>` (specific session)

**Trigger examples:** "retro this session", "复盘一下", "what went wrong this round"

### `/review-backend-arch`

Engineering-framework-level architecture review for .NET backends. Evaluates solution structure, module boundaries, domain partitioning, and dependency direction — **not** implementation details.

- Three review levels: solution, module, boundary
- Seven scoring dimensions, each 0–100
- Always produces a "Do Not Change" section to resist over-architecture
- Manually invoked (not auto-triggered)

### `/tidy-knowledge`

Editorial pass over project documentation at natural pause points (before commit, before merge, end of session, handoff). Reconciles `CLAUDE.md`, `README.md`, and the project's doc surface against the current code.

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
/review-backend-arch
/tidy-knowledge
```

Skills are automatically suggested by Claude when your request matches their trigger conditions — you don't always need to invoke them explicitly.
