# Retro Report Template

When to read this file: at retro Step 4 ("produce the report"). **Follow the skeleton below strictly** — section order, section titles, and field names must not be changed.

## Report skeleton

```markdown
## Retro: <first 8 chars of session-id> — <YYYY-MM-DD> — <one-sentence summary of what this segment/session did>

<If segment-scoped, add on the same line or directly below:>
`[segment N/M — from <clear-timestamp> to end]`
<If full-session or no clears found, omit the segment tag.>

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
- **Apply via**: <update-config skill recommended / manual edit / add to package.json scripts>
- **Exact change**:

  ```<json | bash | jsonc>
  <paste-ready snippet or steps>
  ```

<If nothing applies, omit the entire section.>

### What went well

<2-3 bullets. **Be honest** — only write success patterns you actually observed; do not pad.>

- <e.g., user gave a clear spec, Claude located the right file on first try and edited it correctly, no rework>
- <e.g., plan-mode alignment landed, sub-agents had no failing retries>

### Cutoff notice

<Only when analyzing the current session and the tail may be truncated.>

The last N lines may not yet be flushed; this analysis covers content up to timestamp <last-read timestamp>.

---

*This report is for your review. Tell me if you want any of it applied.*
```

## Full example (synthesized session)

The following is a retro report for a **fictional session**, used only as a style reference. **Do not copy the specific content** — just match the structure and tone.

```markdown
## Retro: a3f1c2b8 — 2026-04-30 — Refactor of Settings page in src/web/components/SettingsPage.tsx
`[segment 2/2 — from 2026-04-30T14:22:10Z to end]`

### TL;DR
- Extracted SettingsPage into a zustand store; UI behavior is correct, all tests green.
- Main loss: the same file was Edit'd 6 times because the plan stage didn't lock down the prop interface.
- Top lesson: cross-component changes need an "interface contract" drawn at plan stage.

### Issues observed

#### 1. SettingsPage.tsx repeated rework (6 Edits)
- **Symptom**: jsonl line 432-1280, six `Edit` calls on `src/web/components/SettingsPage.tsx`, each one overturning the previous prop shape. Final structure roughly matches the first draft.
- **Root cause**: inferred — the plan stage's DESIGN.md did not draw an interface contract. The need for a new prop on the parent component only surfaced after the first Edit, triggering chained changes.
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
- **Apply via**: `update-config` skill (recommended)
- **Exact change**:

  ```jsonc
  // file: .claude/settings.local.json
  // location: append to the permissions.allow array
  "Bash(dotnet test:*)"
  ```

### What went well

- plan-mode aligned the functional behavior; final UI behavior was correct, no functional bugs.
- No mid-token sub-agent truncation; DoD gate and BLOCKED exit both fired correctly.
- Tests passed first try, no flakiness.

---

*This report is for your review. Tell me if you want any of it applied.*
```
