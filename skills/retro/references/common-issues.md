# Common Session Patterns — Issue Catalog + Positive Signals

When to read this file: at retro Step 3 ("cluster issues by type, prioritize, and draft"). Scan the headings first, then return to the transcript with the signatures for each pattern that looks relevant.

The file has two parts:
- **Issue families (§ 1–6)** — anti-patterns to put under "Issues observed"
- **Positive signals (§ end of file)** — success patterns to put under "What went well"

Each issue family includes:
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

## 3. Workflow / convention drift

**Signature**:
- The user stated a specific workflow, agent, or convention earlier in the session ("use the X workflow", "always run the linter first", "dispatch to agent Y for this kind of task") — and Claude bypassed it.
- An agent or sub-agent's output complains about a missing prerequisite artifact (a design doc, test plan, contract, schema) — signaling the agent was invoked outside the workflow it expects.
- A sub-agent retried the same root cause ≥ 3 times with no clean exit (no escalation, no halt, no handoff).
- A sub-agent's output was truncated mid-token — context budget exhausted before the task completed.

**Why it matters**: bypassing the user's stated workflow forces the user to redo the work or accept inferior results. Specialized agents typically assume specific upstream artifacts; invoking them off-piste fails outright or wastes the entire invocation.

**Example diagnosis** (illustrative — your project's agents, doc names, and conventions will differ; substitute your own vocabulary):

> **Symptom**: line 1234-1289, Claude dispatched the `test-implementer` agent (which expects `DESIGN.md` + `TEST-PLAN.md`) to fix a stray lint error. The agent rejected the task with "missing DESIGN.md".
> **Root cause**: workflow-vs-generic distinction not consulted — should have used a general-purpose agent.
> **Severity**: medium (one full agent invocation wasted + needs re-dispatch)
> **Recurrence risk**: likely to repeat

Project-specific role names, doc names, and workflow conventions belong in the project's `CLAUDE.md` / `AGENTS.md`, not in this skill. This skill recognizes the **pattern** (workflow drift); the project provides the **vocabulary**.

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
> **Apply via**: manual edit of `.claude/settings.local.json` (or your project's config-update workflow)

## 6. Avoidable rework

**Signature**:
- Same file `Edit`'d ≥ 3 times (and each pass overturns the previous implementation)
- File was `Write`-ten then deleted (dead-end)
- Steps in TaskCreate / a plan got overturned and re-planned

**Why it matters**: rework is a symptom of **insufficient root-cause exposure** — usually the plan stage didn't align before implementation started. If a session has ≥ 2 rework loops, the report should call out the specific plan gap (no interface contract drawn, no API signature confirmed, etc.) instead of vaguely saying "plan better next time."

---

## Positive signals — for the "What went well" section

When `What went well` is empty or padded with vagueness, the report loses calibration value — every retro reads as "this was a disaster." Look for these concrete success patterns and write them up *with the same evidence discipline as issues*: cite line ranges or tool-call patterns, not adjectives.

### Signatures

- **First-shot file accuracy**: target file located on the first `Read` (no preceding broad `Glob` / `Grep` exploration before opening the right file). Signals project-structure understanding.
- **No-correction phase**: a multi-step task block (e.g., a feature implementation) ran ≥ 10 turns without a user correction or direction reversal. Signals aligned understanding.
- **Plan-implementation match**: the initial `TaskCreate` task list (or plan-mode plan) survived to completion with no `TaskUpdate` to task descriptions and no abandoned items. Signals correct scope assessment up front.
- **Clean sub-agent invocation**: a sub-agent (`Agent` tool) completed without failed retries, mid-token truncation, or "missing prerequisite" complaints. Signals correct agent selection.
- **Efficient tool selection**: used `Grep` for content search instead of multiple `Read`s; used `Glob` for file enumeration instead of recursive `ls`; used `Edit` for surgical changes instead of `Write`-overwrite. Signals tool fluency.
- **Tests passed first try**: test command in `Bash` succeeded on first invocation after implementation, no retries for flakiness.

Write each as an evidence-grounded one-liner — e.g., *"First-shot file location: `Read` of `src/web/Settings.tsx` at line 145 with no preceding search calls"* — not *"Claude understood the structure well."* The latter is unfalsifiable; the former is calibratable.

If you genuinely find no notable positive patterns, write: *"No success patterns worth highlighting — see Issues for what dominated this session."* Honesty preserves the report's signal value over time; padding destroys it.
