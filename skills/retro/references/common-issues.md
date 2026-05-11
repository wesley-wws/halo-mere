# Common Session Issues — Pattern Catalog

When to read this file: at retro Step 3 ("cluster issues by type"). Scan the table of contents first, then come back to the transcript with the signatures for each family that looks relevant.

Each pattern includes:
- **Signature** (how to recognize it in the transcript)
- **Why it matters** (cost of leaving it un-fixed)

## 1. Tool-use errors

**Signature**: `"is_error":true` / `InputValidationError` / same tool retried back-to-back (args tweaked, second attempt succeeds).

**Typical cases**:
- `Grep` regex problems (unescaped braces, forgot `multiline:true`, wrong file type)
- `Read` path errors (drive-letter case, `/` vs `\`, broken relative paths)
- `Edit` `old_string` not unique or includes the line-number prefix
- `Bash` commands using cmd-only syntax or wrong path separators on Windows

**Why it matters**: each retry burns ~1–3k tokens + 1–3 seconds and pollutes context. Late-session grep/read errors are often delayed manifestations of an early mistake.

## 2. Re-reading and context misuse

**Signature**:
- Same file path appears in `Read` calls ≥ 3 times (with no obvious modification in between)
- `Glob` / `Grep` runs once, then re-runs the same pattern
- A `Skill` / `Agent` tool exists that could one-shot it, but multiple `Read`s were used instead

**Why it matters**: one `Read` of a large file is several thousand tokens; doing it three times noticeably shrinks usable context. Long-session "context exhaustion" near the tail usually traces back here.

## 3. Workflow drift

**Signature**:
- User said "use /dev-workflow" or "follow the dev-workflow process", but Claude dispatched a `general-purpose` agent directly
- The reverse: an ad-hoc task got dispatched to a workflow-bound agent (test-implementer / code-implementer / code-reviewer) — the transcript will show the agent complaining about "missing DESIGN.md"
- Sub-agent has no BLOCKED exit; got stuck on the same root cause 3+ times and kept retrying
- Sub-agent output truncated mid-token (context blew up)

**Why it matters**: workflow-bound agents assume a full dev-workflow context (DESIGN.md, TEST-PLAN.md). Using them for ad-hoc tasks fails outright or wastes a full agent invocation.

**Example diagnosis** (write under Issues):

> **Symptom**: line 1234-1289, Claude dispatched test-implementer to fix a stray lint error. Agent rejected with "missing DESIGN.md / TEST-PLAN.md".
> **Root cause**: workflow-vs-generic distinction not consulted.
> **Severity**: medium (one full agent invocation wasted + needs re-dispatch)
> **Recurrence risk**: likely to repeat

## 4. User friction signals

**Signature**:
- Same kind of correction appears ≥ 2 times in the session (user repeating "don't do that")
- User messages get shorter and more terse (clipped sentences, more exclamation marks)
- User repeats the same instruction (Claude didn't hear / misheard)
- User switches between English and Chinese — often because Claude used the wrong language earlier

**Why it matters**: friction is **the strongest issue signal** — anything the user spent effort correcting deserves prominent placement in the report. Default to bumping severity by one level.

## 5. Harness / config gaps

**Signature**:
- Same permission prompt fires ≥ 2 times in the session (the Bash pattern isn't on the allowlist)
- User says "from now on every X do Y" / "whenever..." / "each time..." → this is a hook signal
- Commands written in CLAUDE.md / docs/ failed during the actual session (docs are stale)

**Why it matters**: harness changes apply **once and benefit every future session** — best ROI. Even though retro only suggests and never applies, surfacing the gap makes it visible to the user.

**Example diagnosis** (write under Suggested workflow / config improvements):

> **Change**: add `"Bash(dotnet test:*)"` to the `permissions.allow` array in `.claude/settings.local.json`
> **Trigger frequency observed**: 4 permission prompts this session
> **Apply via**: `update-config` skill (recommended)

## 6. Avoidable rework

**Signature**:
- Same file `Edit`'d ≥ 3 times (and each pass overturns the previous implementation)
- File was `Write`-ten then deleted (dead-end)
- Steps in TaskCreate / a plan got overturned and re-planned

**Why it matters**: rework is a symptom of **insufficient root-cause exposure** — usually the plan stage didn't align before implementation started. If a session has ≥ 2 rework loops, the report should call out the specific plan gap (no interface contract drawn, no API signature confirmed, etc.) instead of vaguely saying "plan better next time."
