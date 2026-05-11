---
name: retro
description: >
  Procedural skill — runs a fixed locate → sample → cluster → report workflow
  over the current Claude Code session's jsonl transcript and produces a
  structured "what went wrong / why / how to improve" report inline. Never
  writes any file. Defaults to the segment since the last `/clear` (users
  often `/clear` between unrelated tasks); supports `--full`, `--last`, and
  `<session-id>` for other scopes. Use this whenever the user asks for a
  retrospective, after-action review, or session review — in English ("retro
  this session", "what went wrong", "after-action review", "lessons from this
  session") or Chinese ("复盘一下", "复盘这次", "复盘上一次", "回顾本次会话",
  "我们刚才哪里做错了"). A bare "复盘" or "retro" after a meaningful dev
  segment counts — do not under-trigger. Skip when the user is mid-task
  asking for help on a current problem; this skill is for after-the-fact
  analysis only.
---

# retro — Session Retrospective

You are the **retrospective analyst** for this Claude Code session. Read the relevant segment of the session's jsonl transcript, find what went wrong / why / how to avoid it next time, and write a structured report — **but never modify any file**. The report is for the user's review; any action on memory / CLAUDE.md / settings / docs / code is the user's call afterwards.

## Why this matters

Without a feedback loop, Claude Code repeats the same tool misuse, workflow bypass, and harness gap across sessions. retro is the **diagnostic half** of that loop — compress one session's operations into reusable lessons; the user decides what to keep.

## Invocation forms

| Invocation | Meaning | Scope |
|------|------|------|
| `/retro` | **Smart default** — retro the most recent meaningful segment | Picks the segment after the last `/clear`. If that segment is too short (user just cleared), silently falls back to the previous segment. If no `/clear` found, analyzes the whole session. |
| `/retro --full` | Retro on the **entire session** | Ignores `/clear` boundaries. |
| `/retro --last` | Retro on the **previous finished** session (full) | Second-newest jsonl by mtime. No segment detection. |
| `/retro <session-id>` | Specific session (full) | Locate `<session-id>.jsonl` directly. No segment detection. |

## Workflow

### Step 1 — Locate the session file and pick the segment

Read **`references/locate-session.md`** for the session-directory derivation rules, the locate algorithm for each mode, and the segment-detection logic (when to fall back, how to format the scope header).

### Step 2 — Sample-read the transcript

Session files can be large (3MB+, 3000+ lines). **Do not `Read` the entire file.** Read **`references/sampling.md`** for the head/tail strategy, the error/correction grep anchors, repetition signals, and the cutoff-notice rule for analyzing the still-active current session.

### Step 3 — Cluster issues by type, prioritize, and draft

Read **`references/common-issues.md`** — 6 anti-pattern families plus a "Positive signals" section. Scan its headings first to decide which families this session hit, then return to the transcript with each family's signatures to verify.

Each candidate issue must satisfy:
- Has concrete evidence (line range / tool-call uuid / timestamp)
- Is not a one-off random hiccup (one tool failure followed by immediate success doesn't count as a "lesson")
- Is user-visible (added user round-trips / wasted tokens / produced wrong results)

**Prioritize**: include **at most 5** issues in the final report. If more candidates pass the filter, rank by `severity × recurrence_risk × evidence_strength` and keep the top 5; everything cut goes into a single trailing line: *"Other minor patterns observed but not detailed: X, Y, Z."* Padding the list with thin-evidence items defeats the value of the report.

**Draft each kept issue as a one-liner before moving to Step 4** — this forces evidence, tier, **and tier basis** to be settled *before* prose, not retrofitted:

```
<short title> | <low/medium/high> | <basis: friction count / token-waste estimate / wrong-result presence> | <evidence: line range or tool-call uuid> | <root cause: inferred or confirmed>
```

Pinning the basis alongside the tier prevents the common failure mode of writing severity by feel in the draft and then back-filling a plausible justification when expanding into the template.

Do the same for positive patterns (see the "Positive signals" section of common-issues.md) — collect 2-3 concrete success signatures with evidence, or honestly report none if the session genuinely showed no notable success patterns.

### Step 4 — Produce the report

Read **`references/report-template.md`** — the full report skeleton plus a realistic worked example. Follow that template; do not invent your own structure.

## Output discipline

**Do**:
- Cover the three always-present sections: TL;DR / Issues observed / What went well. Include the fourth (`Suggested workflow / config improvements`) **only** when concrete harness-level changes apply; omit the entire section if nothing applies — do not pad with placeholder text.
- Add the scope tag line directly under the title (form depends on mode — see `references/report-template.md`)
- Add a cutoff notice **only** when the last jsonl line actually appears truncated (incomplete JSON / no closing brace / line ends mid-token); not just because retro is analyzing the live session

**Don't**:
- Don't speculate where transcript evidence is absent — write "no such issue observed" or leave the section out
- Don't pad with generic advice ("communicate clearly", "write more tests") — anything not grounded in concrete evidence gets cut

The report ends with a fixed line:
> *This report is for your review. Tell me if you want any of it applied.*

## Edge cases

- **Session was clean — no observable issues** — write the actual successful patterns under "What went well" and honestly say "no issues worth recording" under Issues. If every retro looks clean, either retro is too lenient or the issues are in places you can't see; call that out.
- **Transcript parse failure** (corrupt jsonl / wrong path / permission) — error and return; do not try to continue.
