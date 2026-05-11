# Sampling the Transcript

Session files can be 3MB+ and 3000+ lines. **Never `Read` the whole file.** Sample with the order below. In segment mode, constrain all reads and greps to lines within the segment range.

## Small-segment shortcut

If the segment is ≤ 300 lines **and no single line exceeds ~10KB**, `Read` the whole segment (offset + limit) and skip the rest of this file — it fits comfortably in context.

A single jsonl line can be a 50KB+ tool result blob; 300 such lines is megabytes, which busts the context budget. Quick check: `awk '{ print length }' <file> | sort -rn | head -1` — if the longest line exceeds ~10KB, fall through to the sampling order below even when the segment looks short.

## Sampling order

### 1. Head and tail

- **Segment mode**: `Read` the first 50 lines of the segment + the last 200 lines (use offset/limit).
- **Full mode**: same, but on the whole file.

This gives you the start state and final state of the analysis scope.

### 2. Error anchors

`Grep` these patterns, read ±10 lines of context around each match. Discard matches outside the segment range.

- `"is_error":true` — tool returned an error
- `InputValidationError` — tool call args malformed
- `"not found"`, `"file not found"`, `Error: ` — file or content missing
- `permission` and `denied` co-occurring — permission prompt

### 3. User correction anchors

`Grep` the user's correction phrases, read ±5 lines of context:

- **English**: `don't`, `stop`, `no not`, `wait`, `actually`, `we already`, `again`
- **Chinese**: `不对`, `别`, `等等`, `不是`, `刚才`, `又错了`

### 4. Repetition signals

- Same file path's `Read` / `Edit` calls (≥ 3 is suspicious)
- Same tool's adjacent retries

### 5. Session metadata

Extract `sessionId`, `cwd`, `gitBranch` from the head of the file (always read this, even in segment mode — it's session-level context). Confirm the last timestamp from the tail.

### 6. Plan / direction drift anchors

These feed family 6 (avoidable rework). Without them, the family's signature is invisible to grep — direction changes don't always surface as tool errors.

- `TaskCreate` followed later by `TaskUpdate` calls — check whether the update changed the task **description** (not just status). Description changes signal scope creep or mid-stream re-planning.
- Two `TaskCreate` calls within the same segment whose task lists overlap — the earlier list was likely abandoned.
- User messages containing direction reversal, read ±10 lines around each match:
  - **English**: `actually`, `wait`, `let's instead`, `nevermind`, `scratch that`, `forget that`, `change of plan`
  - **Chinese**: `换个思路`, `先别做`, `重新来`, `等下`, `不做这个`, `重新想`

## Cutoff notice for the live session

If you're analyzing the **current session**, the last few lines may still be in the OS write buffer. Inspect the final line: if it is clearly truncated (incomplete JSON, no closing brace, line ends mid-string), add a cutoff notice in the report — see `report-template.md` for the exact rule and banner format.

If the final line parses cleanly, **do not add the banner**. Adding it unconditionally on live sessions trains the user to ignore it (cry wolf). And **do not retry or wait** — read once, decide based on what you got.
