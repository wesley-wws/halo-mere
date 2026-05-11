---
name: tidy-knowledge
description: >
  Editorial pass over project documentation at natural pause points — review
  CLAUDE.md, README.md, and docs/ against the code, reconcile drift, prune
  cruft, surface structural problems. MUST trigger when the user signals a
  pause / handoff / milestone moment: "sync up", "tidy docs", "clean up docs",
  "before commit", "before merge", "ready to ship", "ready for handoff",
  "/sync", "/neat", "整理文档", "整理一下", "梳理一下", "同步一下", "收尾",
  "这个阶段做完了", "准备提交", "要 push 了", "merge 之前", "明天有人接手",
  or any phrase indicating "stop adding code; make sure what's written about
  it is still true." Bare "整理" / "tidy" with prior dev context counts —
  do not under-trigger. DO NOT trigger for code-level cleanup such as
  "refactor function", "tidy imports", "format file", "整理代码",
  "整理 imports", "格式化". Personal-scope state (memory, global config) is
  out of scope; this skill only touches project-scope files.
---

# Tidy Knowledge — Reconcile Project Writing with Code Reality

## What and why

When a developer reaches a pause point — end of session, before commit, before merge, before handoff, after a milestone — written materials about the project drift behind the code. This skill performs an editor's pass: reviewing what is written, comparing it to what the project actually *is*, and reconciling the drift.

You are an **editor, not a logger**. A logger appends; an editor reviews globally — merging duplicates, pruning stale content, fixing inaccuracies, recognizing when structure itself needs to change. Code can be regenerated; rotted docs cannot. Every reader hitting a stale doc pays the cost of decoding it against current code — multiplied across teammates, downstream consumers, and future Claude sessions, that cost dwarfs a single editor's pass. Skipping doesn't save time; it defers a larger cost to readers.

## Two surfaces, two audiences

| Surface | Reader | Job |
|---|---|---|
| `CLAUDE.md` (root + nested) | The next Claude session in this repo | Conventions, structure, gotchas, command lists, route lists — the navigational map for an agent picking up where you left off |
| `README.md` + the project's documentation body (located via CLAUDE.md's inventory; see step 1) | Humans and agents outside this session — teammates, downstream consumers, future maintainers | Onboarding, architecture, operations, handoff |

These surfaces serve **different readers**, so they hold **different content** even when describing the same code. CLAUDE.md tells the next agent "five new routes were added; see the integration guide for what they expose." docs/ tells the downstream consumer how to consume them. Both must exist; neither replaces the other.

The most common failure is conflating them — pasting external-facing prose into CLAUDE.md, or stuffing AI-only context into a public README. Editorial discipline keeps each surface honest about who reads it.

## Four views of any capability

A complete public-facing documentation surface for a feature / API / flow covers four views. Filenames vary across projects; **map them from the doc inventory discovered in step 1**:

1. **Use** — how to invoke it (curl, SDK, error codes). Common names: `integration-guide`, `external-api`, `setup-guide`, `onboarding`, sometimes `README` itself.
2. **Work** — how it works internally (data flow, state machines, design tradeoffs). Common names: `architecture`, `design`, `overview`.
3. **Operate** — how to run and troubleshoot (smoke commands, runbook entries, env vars). Common names: `runbook`, `operator-guide`, `deployment`.
4. **Change** — what shipped (release notes, handoff). Common names: `handoff`, `CHANGELOG`, `RELEASES`.

Not every project has every view. If a change needs a view that doesn't exist, **suggest creating it in your summary; do not auto-create** — orphaned docs that nobody owns are worse than no doc.

## Editing principles

- **Edit, don't append.** New information replaces old. Modify the existing line; don't add a parallel one.
- **Prune freely.** Resolved plans, abandoned decisions, transient context — delete them. The doc is not a journal.
- **Use absolute dates.** "Today", "recently", "last week" become broken records the instant time passes. Write `2026-04-30`.
- **Match audience to surface.** Do not address the next Claude in a public README. Do not stuff AI-only context into a doc humans will read.
- **Project scope only.** Do not modify personal-scope state — memory, `~/.claude/CLAUDE.md`, anything outside the repo. If you notice an issue there, mention it in your summary for the user to address.
- **Edit content; don't author structure.** Updating an existing route table, env var list, or paragraph to match code is content — that is what this skill does. Adding a brand-new top-level section to CLAUDE.md, or creating a doc file that did not exist before, is structure — that belongs to the user. Surface structural changes as Recommendations; do not make them.

## Recognize when structure is the problem

If the same gap recurs across multiple sync passes — the same file forgotten, the same fact repeatedly homeless, the same doc playing two roles for two audiences — that is a **structural signal**, not a one-off oversight.

When you see this pattern, **stop patching**. Surface it in the summary as a structural recommendation: split the dual-purpose doc, create a missing category, consolidate scattered facts. Let the user decide whether to restructure. Patching the same hole indefinitely is busywork; restructuring closes the hole.

## Working method

1. **Discover the doc surface from CLAUDE.md.** The project's `CLAUDE.md` (root + nested) is the canonical source for *what counts as documentation* in this project. Read every `CLAUDE.md` first.

   Find the **doc inventory** by content shape, not by header name. A section qualifies as inventory if it contains any of:
   - Markdown links to in-repo `.md` / `.rst` / `.adoc` files
   - A table mapping topics to file paths (e.g., "When you are X, read `docs/Y.md`")
   - A list of `[Title](path/to/doc.md)` entries
   - References to in-repo directories that hold docs (e.g., "see `docs/`", "runbooks live under `ops/runbooks/`")

   Section titles vary (`Documentation Map`, `Conventions & Knowledge Base`, anything semantically equivalent) — **match by content shape, not by title**.

   Aggregate every distinct path / link / directory across all `CLAUDE.md` files (resolve relative paths against each file's location). Add the always-implicit set: `README.md` and every `CLAUDE.md` itself. The result is the **doc surface** for this pass. Read each file in it.

   **External links** (http / https) → recognize as out-of-repo references; do not edit, but if this session's changes might affect them, note in the summary.

   **Stale pointers** (inventory references a file or directory that no longer exists) → add to Recommendations: "doc inventory at `<path>` references `<missing>` — fix or remove."

   **No inventory found in any `CLAUDE.md`** → fall back to a best-effort scan: root-level `*.md`, plus `docs/` / `documentation/` / `doc/` if any exists. Use this fallback set for the pass; the editorial work still produces value. Then make this the **highest-priority Recommendation** in the summary, with a paste-ready example built from the files you actually discovered, e.g.:

   ```
   ## Documentation Map  (any title works — content shape matters more than name)
   | When you are... | Read |
   |---|---|
   | <topic inferred from <discovered_file_1>> | <discovered_file_1> |
   | <topic inferred from <discovered_file_2>> | <discovered_file_2> |
   ```

   **Do not auto-write the inventory section** — adding a new top-level section to CLAUDE.md is a structural decision that belongs to the user (see Editing principles).

2. **Inventory the changes.** Re-read the conversation. Find: new or removed dependencies, renamed commands, new env vars / routes / tables / SDK surfaces, decisions that overturned earlier ones, **cross-project surface changes** (especially when project A depends on project B — both sides need updating), and user feedback or preferences expressed in passing.

3. **Map changes to surfaces and views.** A new route affects CLAUDE.md's route list AND the Use view AND the Work view. A new env var affects CLAUDE.md AND the Operate view. Walk through each change deliberately; do not eyeball.

4. **Edit, docs first.** External-facing files first (their drift hurts most), then CLAUDE.md. If interrupted partway, readers still see consistent state.

5. **Verify alignment.** Walk through everything once more. Do CLAUDE.md's command lists still match the actual scripts? Does the README setup still work? Did each change land in every view it needed? Run Grep for relative-time markers (`今天|昨天|刚刚|最近|today|yesterday|recently`) — they should all be zero. **If a verification fails three times and you do not have what you need to fix it, list it under Open Questions and move on.** No infinite loops.

6. **Summarize.** Concise report:
   - **Changed** — files you actually edited, grouped by project, one line each describing what changed.
   - **Open questions** — specific decisions you deferred to the user (conflicting evidence, missing information, external dependencies you could not resolve).
   - **Structural recommendations** *(only when something recurring was observed)* — patterns suggesting the doc structure itself should change.

   Skip empty sections. Do not pad.

## Out of scope (mention in summary if relevant; never act)

- **Personal-scope state** (memory, `~/.claude/CLAUDE.md`) — see Editing principles.
- **Authoring from blank** — this skill reconciles existing material; generating a new doc from scratch is a different activity.
- **Code-level cleanup** — refactoring, formatting, import tidying live in other skills.
