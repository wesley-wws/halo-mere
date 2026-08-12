# Retro Report Template

When to read this file: at retro Step 4 ("produce the report"). **Follow the skeleton below strictly** — section order, section titles, and field names must not be changed.

## Report skeleton

```markdown
## Retro: <first 8 chars of session-id> — <YYYY-MM-DD> — <one-sentence summary of what this scope did>

`[<scope tag — see "Scope tag format" below>]`

> **Cutoff notice** (include only when the last jsonl line is truncated — see "Cutoff notice — when to add, when not to" below for the rule):
> The session file's tail appears unflushed; this analysis covers content up to `<last-complete-timestamp>`. The final tool call may not yet be visible.

### TL;DR
- <2-4 bullets. First bullet: what got done. Remaining bullets: the worst issue + the single most worth-remembering lesson.>

### Issues observed

#### <N>. <one-sentence title>
- **Symptom**: <what showed up. Cite concrete evidence: jsonl line N-M / tool-call uuid / timestamp.>
- **Root cause**: <why it happened. Mark explicitly as "inferred" or "confirmed".>
- **Severity**: <low | medium | high> (basis: user friction count / wasted-token estimate / whether it produced wrong results)
- **Recurrence risk**: <one-off | likely to repeat>

<If this session has nothing worth recording, write: "No issues worth recording — see What went well.">

### Suggested workflow / config improvements

<Harness-level (settings, hooks, scripts) suggestions:>

#### <N>. <one-sentence change description>
- **Trigger frequency observed**: <times seen this session / per-occurrence cost estimate>
- **Apply via**: pick ONE based on change type (see "Apply via — decision guide" below)
- **Exact change**:

  ```<json | bash | jsonc>
  <paste-ready snippet or steps>
  ```

<If nothing applies, omit the entire section.>

### What went well

<2-3 bullets. **Be honest** — only write success patterns you actually observed; do not pad.>

- <e.g., user gave a clear spec, Claude located the right file on first try and edited it correctly, no rework>
- <e.g., plan stage aligned the contract before implementation; no sub-agent retries>

---

*This report is for your review. Tell me if you want any of it applied.*
```

## Scope tag format

The `[<scope tag>]` line directly under the title tells the user what slice of work was actually analyzed. The session-id 8-char prefix is already in the title, so it does not need to be repeated here. Use one of these forms based on mode:

| Mode | Scope tag form |
|---|---|
| Default `/retro` (segment found) | `[segment N/M — from <clear-timestamp> to end]` |
| Default `/retro` (no clears found) | `[full — <first-timestamp> to <last-timestamp>]` |
| `/retro --full` | `[full session — <first-timestamp> to <last-timestamp>]` |
| `/retro --last` | `[previous session — <first-timestamp> to <last-timestamp>]` |
| `/retro <session-id>` | `[specified session — <first-timestamp> to <last-timestamp>]` |

**Optional branch suffix**: when `gitBranch` was successfully extracted from the head of the jsonl, append ` · branch: <branch-name>` to any of the above. This gives the user a quick anchor for "which branch was this work on" — especially useful when comparing multiple retros side-by-side.

This line is non-negotiable. The user needs to know what slice of work this report covers — especially when default-mode auto-fallback selected a non-obvious segment.

## Severity tiers — rubric

Severity should be defensible against this rubric, not vibes. Map evidence to tier:

| Tier | Threshold (any one suffices) |
|---|---|
| **low** | Pure friction; no wrong results; estimated token waste < ~3k; the user didn't notice or didn't complain |
| **medium** | Wasted ≥ ~5k tokens OR caused ≥ 2 user round-trips OR produced wrong results that were caught and corrected in-session |
| **high** | Produced wrong results that shipped (weren't caught in-session) OR blocked progress ≥ 15 minutes OR caused the user to abandon and restart a task |

If unsure between two tiers, prefer the lower one — it's easier for the user to disagree and bump up than to disagree and bump down. Severity inflation kills the cross-session calibration value of retro.

### Estimating token waste when the jsonl lacks per-call counts

The jsonl usually does not include per-call token counts. Estimate by order-of-magnitude: typical tool responses ~0.5–2k tokens; large file `Read`s ~2–5k. Use ranges, not point estimates — *"~5–10k tokens wasted on 3 redundant Reads of SettingsPage.tsx"* is honest; *"12k tokens wasted"* is false precision.

## Apply via — decision guide

The `Apply via` field on each suggested change should pick exactly one of these, based on the change shape:

- **`manual edit of <file>`** — single-point config change. Examples: append to `permissions.allow` in `.claude/settings.local.json`; add a rule to `CLAUDE.md` / `AGENTS.md`; add a path to `.gitignore`.
- **`add to a script / package.json scripts / Makefile target`** — operation that should be repeatable or runnable on demand. Examples: regenerating a schema, running a codegen step, payload for a pre-commit hook.
- **`your project's config-update workflow`** — when the project has a dedicated skill / agent / script for config changes; route the user through that rather than touching files directly.

Pick one. Listing multiple options on the same change forces the user to make the routing decision retro should have made.

## Cutoff notice — when to add, when not to

**Add it** when the last line of the jsonl file is clearly truncated: incomplete JSON object, no closing brace, line ends mid-string. The OS may not have flushed the buffer; the final operation isn't fully recorded.

**Don't add it** just because retro is analyzing the live session. If the last line parses cleanly, the data is complete enough — adding the banner anyway is crying wolf and will train the user to ignore it.

## Full example (synthesized session)

The following is a retro report for a **fictional session**, used only as a style reference. **Do not copy the specific content** — just match the structure and tone.

```markdown
## Retro: a3f1c2b8 — 2026-04-30 — Refactor of Settings page in src/web/components/SettingsPage.tsx
`[segment 2/2 — from 2026-04-30T14:22:10Z to end · branch: feat/settings-refactor]`

### TL;DR
- Extracted SettingsPage into a zustand store; UI behavior is correct, all tests green.
- Main loss: the same file was Edit'd 6 times because the plan stage didn't lock down the prop interface.
- Top lesson: cross-component changes need an "interface contract" drawn at plan stage.

### Issues observed

#### 1. SettingsPage.tsx repeated rework (6 Edits)
- **Symptom**: jsonl line 432-1280, six `Edit` calls on `src/web/components/SettingsPage.tsx`, each one overturning the previous prop shape. Final structure roughly matches the first draft.
- **Root cause**: inferred — the plan stage's design doc did not draw an interface contract. The need for a new prop on the parent component only surfaced after the first Edit, triggering chained changes.
- **Severity**: medium (no functional bug, but ~12k tokens wasted + 15 minute delay)
- **Recurrence risk**: likely to repeat

#### 2. dotnet test triggered permission prompts 4 times
- **Symptom**: jsonl lines 215, 678, 1100, 1450 each show "permission required for Bash(dotnet test ...)".
- **Root cause**: confirmed — `permissions.allow` in `.claude/settings.local.json` lacks a `Bash(dotnet test:*)` pattern.
- **Severity**: low (no functional impact, pure friction)
- **Recurrence risk**: likely to repeat

### Suggested workflow / config improvements

#### 1. Add `Bash(dotnet test:*)` to the allowlist
- **Trigger frequency observed**: 4 permission prompts this session
- **Apply via**: manual edit of `.claude/settings.local.json`
- **Exact change**:

  ```jsonc
  // file: .claude/settings.local.json
  // location: append to the permissions.allow array
  "Bash(dotnet test:*)"
  ```

### What went well

- plan-mode aligned the functional behavior; final UI behavior was correct, no functional bugs.
- No mid-token sub-agent truncation; sub-agents halted cleanly on their own gates.
- Tests passed first try, no flakiness.

---

*This report is for your review. Tell me if you want any of it applied.*
```
