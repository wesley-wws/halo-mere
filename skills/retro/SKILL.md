---
name: retro
description: >
  Retrospective on a just-finished Claude Code session or segment — read the
  session's jsonl transcript from disk, identify what went wrong / why / what
  to improve, and output a structured report. Defaults to the current segment
  (work since the last /clear) rather than the entire session, because users
  often /clear between unrelated tasks. Produces a report ONLY — never modifies
  any file. MUST trigger when the user says: "/retro", "复盘这次", "复盘上一次",
  "回顾这次 session", "回顾本次会话", "总结一下问题", "retro this session",
  "retrospective", "after-action review", "review this session", "what went wrong",
  "我们刚才哪里做错了", "复盘一下", or any phrase asking for a self-assessment
  of the just-finished work — especially after a milestone, before commit, or
  when the user wants to capture lessons before they're forgotten. Bare "复盘"
  / "retro" with prior dev context counts — do not under-trigger. DO NOT
  trigger when the user is mid-task asking for help with a current problem;
  this skill is for after-the-fact analysis only.
---

# retro — Session Retrospective

You are the **retrospective analyst** for this Claude Code session, not a decision-maker. Read the transcript (or the relevant segment of it), find what went wrong, why, and how to avoid it next time, and write up a structured report for the user — but **never modify any file**.

> **Scope statement**: this skill **only produces a report**. Any change to memory / CLAUDE.md / settings / docs / code is the user's call afterwards. This is a hard constraint — single responsibility, no scope creep.

## Why this matters

Claude Code tends to repeat the same mistakes across sessions — same tool misuse, same workflow bypassed, same harness config gap that makes Claude prod the same hole again and again. The reason is there's no **systematic feedback loop** carrying "what we learned last session" into "next session."

retro is the **diagnostic half** of that loop: compress one session's actual operations into reusable lessons, and let the user decide which ones are worth keeping. Over time, the probability of repeated errors drops noticeably.

## Invocation forms

| Invocation | Meaning | Scope |
|------|------|------|
| `/retro` | **Smart default** — retro the most recent meaningful segment | Picks the last segment (after the last `/clear`). If that segment is too short (you just cleared), silently falls back to the previous segment. If no `/clear` found, analyzes the entire session. |
| `/retro --full` | Retro on the **entire session** | Ignores `/clear` boundaries; analyzes every line in the jsonl. |
| `/retro --last` | Retro on the **previous finished** session (full) | In session dir, pick the second-newest jsonl by mtime. No segment detection. |
| `/retro <session-id>` | Specific session (full) | Locate `<session-id>.jsonl` directly. No segment detection. |

Natural-language triggers without a slash ("retro this one", "复盘一下这次") behave as `/retro` (smart default).

## Workflow

### Step 1: Locate the session jsonl file and determine segment boundaries

**How the session directory path is derived:**

1. Claude's config root is typically `~/.claude` (standard install) or `~/.claude-console` (some distributions).
   If the global CLAUDE.md explicitly specifies one (e.g. `Your config directory is ~/.claude-console`), use that.
2. Session dir = `<config-root>/projects/<encoded-cwd>/`
   Encoding rule from current `pwd`: drive-letter colon (`:`) → `--`, path separators (`\` or `/`) → `-`.
   Examples:
   - cwd = `/home/user/proj/my-app` → `home-user-proj-my-app`
   - cwd = `D:\Code\MyApp` → `D--Code-MyApp`

Each file is `<session-uuid>.jsonl`. Sub-agent threads live in `subagents/agent-*.jsonl` — **skip these**, they are not main sessions.

**Default mode (no args)**:
1. Prefer environment variable `$CLAUDE_SESSION_ID` to construct the file path directly.
2. Otherwise `ls -t *.jsonl | head -5`, then `head -1` each candidate and compare the `cwd` field; the first match is the current session.
3. If neither works, tell the user: "Cannot identify the current session — please use `/retro --last` or `/retro <id>`", then stop.

**`--last` mode**: `ls -t *.jsonl | head -5`, exclude the current sessionId, pick the first.

**Specific id**: read `<session-id>.jsonl` directly; error if missing.

#### Segment detection (applies to `/retro` only)

Users often `/clear` between unrelated tasks within one session. `/retro` automatically picks the most recent segment that's worth analyzing. `--full`, `--last`, and `<session-id>` modes skip segment detection entirely.

1. **Find clear boundaries**: `grep -n 'command-name>/clear<' <file>` — this matches the `/clear` command as recorded in the jsonl. Each match is a segment boundary. Count the matches to get total segments = matches + 1.
2. **Pick the best segment** (no user flags needed):
   - Start with the **last segment** (after the last `/clear`).
   - If that segment is too short (< 5 turns or < 100 lines) — the user probably just cleared — **silently fall back** to the previous segment (between the second-to-last clear and the last clear; or from file start to first clear if only one clear exists).
   - If no clears found, analyze the entire file.
   - If all segments are too short, tell the user: "No segment has enough content to retro — try `/retro --full`."
3. **Add a scope line to the report header**: `[segment N/M — from <clear-timestamp> to <end-timestamp-or-"end">]` where M = total segments (clear count + 1), N = which segment is being analyzed. Extract the timestamp from the `/clear` event's `"timestamp"` field in the jsonl. This tells the user exactly what was analyzed — especially important when auto-fallback selected a non-obvious segment.

### Step 2: Sample-read the transcript

The session file can be large (3MB+, 3000+ lines). **Do not `Read` the entire file** — sample in this order. When operating in segment mode, constrain all reads and greps to lines within the segment range.

**Small segment shortcut**: if the segment is ≤ 300 lines, just `Read` the whole segment (offset + limit) and skip the sampling strategy below — it fits comfortably in context.

1. **Head and tail**: In segment mode, `Read` the first 50 lines of the segment + last 200 lines (use offset/limit). In full mode, read the first 50 lines of the file + last 200 lines. This gives you the start state + final state of the analysis scope.
2. **Error anchors**: `Grep` the patterns below, read ±10 lines of context around each match (discard matches outside the segment range):
   - `"is_error":true` — tool returned an error
   - `InputValidationError` — tool call args malformed
   - `"not found"`, `"file not found"`, `Error: ` — file or content missing
   - `permission` and `denied` co-occurring — permission prompt
3. **User correction anchors**: `Grep` the user's correction phrases, read ±5 lines of context:
   - English: `don't`, `stop`, `no not`, `wait`, `actually`, `we already`, `again`
   - Chinese: `不对`, `别`, `等等`, `不是`, `刚才`, `又错了`
4. **Repetition signals**: count the same file path's `Read` / `Edit` calls (≥ 3 is suspicious); same tool's adjacent retries.
5. **Session metadata**: extract `sessionId`, `cwd`, `gitBranch` from the head of the file (always read this, even in segment mode — it's session-level context); confirm the last timestamp from the tail.

If you're analyzing the **current session**, the last few lines may still be in the OS write buffer — what you read might be incomplete. Add a cutoff notice in the report stating the last timestamp covered, **do not retry or wait**.

### Step 3: Cluster issues by type

Read `references/common-issues.md` — it contains 6 common anti-pattern families with transcript signatures and why they matter. **Scan the table of contents first** to decide which families this session hit, then go back to the transcript with each family's signature to verify.

Each candidate issue must satisfy:
- Has concrete evidence (transcript line range / tool-call uuid / timestamp)
- Not a one-off random hiccup (one tool failure followed by immediate success doesn't count as a "lesson")
- Is user-visible (added user round-trips / wasted tokens / produced wrong results)

### Step 4: Produce the report

Read `references/report-template.md` — it has the full report skeleton and a realistic worked example. **Follow that template strictly**, do not invent your own structure.

## Output discipline

**Do**:
- Cover all four sections: TL;DR / Issues observed / Suggested workflow & config / What went well
- Tie every issue to specific transcript evidence (line range or tool-call uuid or timestamp)
- Add a cutoff notice when the tail may have been truncated

**Don't**:
- Don't write any file (memory / CLAUDE.md / settings / docs / source code)
- Don't speculate where transcript evidence is absent (write "no such issue observed" or leave the section out)
- Don't pad with generic advice ("communicate clearly", "write more tests" — anything not grounded in concrete evidence gets cut)

The report ends with a fixed line:
> *This report is for your review. Tell me if you want any of it applied.*

## Edge cases

**Segment / session too short to retro**: handled by the auto-fallback in segment detection step 2 — silently picks the previous segment; if all segments are too short, suggests `--full`.

**Session was clean — no observable issues**: write the actual successful patterns under What went well, and honestly say "no issues worth recording" under Issues. This should be rare; if every retro looks clean, either retro is too lenient or the issues are in places the user can't see — call that out.

**Transcript parse failure** (corrupt jsonl / wrong path / permission): error and return, don't try to continue.
