---
name: tidy-knowledge
description: >
  Editorial pass over project documentation at natural pause points — review
  CLAUDE.md / AGENTS.md, README.md, and docs/ against the code, reconcile drift, prune
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

You are an **editor, not a logger**. A logger appends; an editor reviews globally — merging duplicates, pruning stale content, fixing inaccuracies, recognizing when structure itself needs to change. Code can be regenerated; rotted docs cannot. Every reader hitting a stale doc pays the cost of decoding it against current code — multiplied across teammates, downstream consumers, and future agent sessions, that cost dwarfs a single editor's pass. Skipping doesn't save time; it defers a larger cost to readers.

## Two surfaces, two audiences

| Surface | Reader | Job |
|---|---|---|
| `CLAUDE.md` / `AGENTS.md` (root + nested; either or both) | The next agent picking up this repo | Conventions, structure, gotchas, command lists, route lists — the navigational map for whoever (or whatever) walks in next |
| `README.md` + the project's documentation body (located via the inventory in `CLAUDE.md` / `AGENTS.md`; see step 1) | Humans and agents outside this session — teammates, downstream consumers, future maintainers | Onboarding, architecture, operations, handoff |

These surfaces serve **different readers**, so they hold **different content** even when describing the same code. The agent-facing file (`CLAUDE.md` / `AGENTS.md`) tells the next agent "five new routes were added; see the integration guide for what they expose." The public-facing docs tell the downstream consumer how to consume them. Both surfaces must exist; neither replaces the other.

The most common failure is conflating them — pasting external-facing prose into the agent-facing file, or stuffing AI-only context into a public README. Editorial discipline keeps each surface honest about who reads it.

## Up to four angles on any capability

A capability can be documented from up to four angles. Few projects need all four — match angles to the project, not the project to angles:

| Angle | Question | Common locations (filenames vary; map from step 1's inventory) | Skip when |
|---|---|---|---|
| **Use** | How does a consumer invoke it? | API: `integration-guide`, `external-api`. CLI: command reference, `--help` output. Library: API docs. Plugin: invocation pattern in README. | Almost never — every consumable thing needs a use story somewhere. |
| **Work** | How does it work inside? | `architecture`, `design`, `overview`, ADRs. | Trivially small projects where the code is the design. |
| **Operate** | How is it run and troubleshooted? | `runbook`, `operator-guide`, `deployment`, env-var inventory. | Anything not running as a long-lived service — libraries, plugins, scripts. |
| **Change** | What just shipped? | `CHANGELOG`, release notes, `handoff`. | Solo or personal projects with no outside consumers. |

A pure library might have only Use + Work. A personal plugin might have only Use. **Don't fabricate views the project doesn't need; do flag a genuinely missing view as a Recommendation** — orphaned docs that nobody owns are worse than no doc.

## Editing principles

- **Edit, don't append.** New information replaces old. Modify the existing line; don't add a parallel one.
- **Prune freely.** Resolved plans, abandoned decisions, transient context — delete them. The doc is not a journal.
- **Use absolute dates.** "Today", "recently", "last week" become broken records the instant time passes. Write `2026-04-30`.
- **Match audience to surface.** Do not address the next agent in a public README. Do not stuff AI-only context into a doc humans will read.
- **Project scope only.** Do not modify personal-scope state — memory, `~/.claude/CLAUDE.md`, anything outside the repo. If you notice an issue there, mention it in your summary for the user to address.
- **Edit content; don't author structure.** Updating an existing route table, env var list, or paragraph to match code is content — that is what this skill does. Adding a brand-new top-level section to the agent-facing file, or creating a doc file that did not exist before, is structure — that belongs to the user. Surface structural changes as Recommendations; do not make them.

## Recognize when structure is the problem

Within this pass, watch for these patterns: a fact that has no clear home (you keep wanting to put it somewhere but no file is the right fit), a single doc playing two roles for two audiences (e.g., a README that contains both onboarding prose and AI-only context), or scattered fragments of the same concept across files that should consolidate. These are **structural signals**, not one-off oversights — you can identify them from a single pass. If the user mentions that the same pattern has recurred across prior passes, treat that as extra evidence, not as a prerequisite.

When you see this, **stop patching**. Surface it in the summary as a structural recommendation: split the dual-purpose doc, create a missing category, consolidate scattered facts. Let the user decide whether to restructure. Patching the same hole indefinitely is busywork; restructuring closes the hole.

## Working method

1. **Precondition: find the agent-facing file.** This skill anchors on `CLAUDE.md` and/or `AGENTS.md` (root + any nested instances). Look for either or both.

   **If neither exists anywhere in the project, halt.** Tell the user: "This skill reconciles documentation against the project's agent-facing file (`CLAUDE.md` or `AGENTS.md`). That file defines the audience for one of the two surfaces and (usually) points to the other. Without one, there is no anchor for a tidy pass. Create a `CLAUDE.md` or `AGENTS.md` first — even a minimal one listing the project's conventions — and then re-run." Do not fall back to a blind scan; the editorial framing depends on this anchor.

   **If found, read every instance.** `CLAUDE.md` and `AGENTS.md` are parallel agent-facing surfaces — both subject to this pass. If both exist with overlapping content, flag as a Recommendation (consolidate or symlink); do not pick a winner yourself. If they **conflict on a specific point** (not just overlap — e.g., one says "use pnpm" and the other says "use yarn"), do not resolve it either — surface as an **Open Question** in the summary, with both excerpts quoted so the user can decide.

   **Extract the doc inventory by content shape, not by header name.** A section qualifies as inventory if it contains any of:
   - Markdown links to in-repo `.md` / `.rst` / `.adoc` / `.txt` files
   - A table mapping topics to file paths (e.g., "When you are X, read `docs/Y.md`")
   - A list of `[Title](path/to/doc.md)` entries
   - References to in-repo directories that hold docs (e.g., "see `docs/`", "runbooks live under `ops/runbooks/`")

   Section titles vary (`Documentation Map`, `Conventions & Knowledge Base`, anything semantically equivalent) — **match by content shape, not by title**.

   Aggregate every distinct path / link / directory across all agent-facing files (resolve relative paths against each file's location). Add the always-implicit set: `README.md` and every agent-facing file itself. The result is the **doc surface** for this pass. Read each file in it.

   **External links** (http / https) → recognize as out-of-repo references; do not edit, but if this session's changes might affect them, note in the summary.

   **Stale pointers** (inventory references a file or directory that no longer exists) → add to Recommendations: "doc inventory at `<path>` references `<missing>` — fix or remove."

   **Agent-facing file exists but contains no inventory** → use the always-implicit set (`README.md`, the agent-facing files themselves, and any obvious `docs/` / `documentation/` directory) as the working surface. Then make this the **highest-priority Recommendation** in the summary, with a paste-ready example built from the files you actually discovered:

   ```
   ## Documentation Map  (any title works — content shape matters more than name)
   | When you are... | Read |
   |---|---|
   | <topic inferred from <discovered_file_1>> | <discovered_file_1> |
   | <topic inferred from <discovered_file_2>> | <discovered_file_2> |
   ```

   **Do not auto-write the inventory section** — adding a new top-level section to an agent-facing file is a structural decision that belongs to the user (see Editing principles).

2. **Discover what changed.** Two signal sources. Use whichever has signal — in most real passes both, in cold invocation only (b):

   **a. Conversation history.** If this session did the work, the conversation is the primary signal. Read it for: new or removed dependencies, renamed commands, new env vars / routes / tables / SDK surfaces, decisions that overturned earlier ones, and user preferences expressed in passing.

   **b. Git state.** When the conversation is thin (the skill was invoked at the start of a session, or to do a periodic cleanup) or you want to corroborate it (changes may predate this session), reach for git:
   - `git status` + `git diff` — uncommitted changes
   - `git diff --staged` — staged changes
   - For "before merge" triggers: `git log <main>...HEAD` and `git diff <main>...HEAD` (substitute the project's main branch name)
   - For "before handoff" / cold cleanup: `git log -20`, or `git log --since=<date the docs were last edited>` to scan accumulated commits

   If neither source yields anything (cold invocation, clean working tree, no recent commits), ask the user what the pass should focus on rather than guessing. If the project is not a git repo, conversation history is the only source — fall back to asking the user when it's empty.

   **Single-repo scope.** Edit only the current repository. If you observe that changes likely break documentation in a dependent repo (e.g., a backend route rename invalidating the frontend's documented contract), surface in Recommendations — do not switch contexts.

3. **Map changes to surfaces and angles.** Each change touches multiple places. Adapt examples to your project's actual angles — don't chase angles that don't exist:
   - **Service example:** a new route → agent-facing file's route list + Use angle + Work angle. A new env var → agent-facing file + Operate angle.
   - **CLI example:** a new subcommand → command reference + `--help` output + agent-facing file's command list.
   - **Library / plugin example:** a new public surface → Use angle (signature / invocation pattern) + Work angle (if design changed) + agent-facing file's surface list.

   Walk through each change deliberately; do not eyeball.

4. **Edit in this order: external-facing first, then agent-facing.** External-facing drift hurts the most readers, so close it first. If interrupted partway, readers still see consistent state.

5. **Verify alignment.** Walk through everything once more. Do the agent-facing file's command lists still match the actual scripts? Does the README's setup section still match the project's actual scripts and commands (compare **statically** — don't execute setup steps)? Did each change land in every angle it needed? Use the **Grep tool** (or `grep -E`) to search for relative-time markers — the pattern uses regex alternation, so it needs a tool that supports it: `今天|昨天|刚刚|最近|现在|目前|近期|前几天|上周|today|yesterday|recently|currently|lately|this week|next week`. Hits should be zero. **If you can't resolve a verification after a couple of tries and don't have the information to fix it, list it under Open Questions and move on.** No infinite loops.

6. **Summarize.** Concise report:
   - **Changed** — files you actually edited, one line each describing what changed.
   - **Open questions** — specific decisions you deferred to the user (conflicting evidence, missing information, external dependencies you could not resolve).
   - **Structural recommendations** *(only when a structural pattern is genuinely visible — see "Recognize when structure is the problem"; don't pad with speculation)* — patterns suggesting the doc structure itself should change.

   Skip empty sections. Do not pad.

## Out of scope (mention in summary if relevant; never act)

- **Personal-scope state** (memory, `~/.claude/CLAUDE.md`) — see Editing principles.
- **Sibling repositories.** This skill edits only the current repo; cross-repo doc drift goes in Recommendations (see step 2).
- **Non-text documentation** (Word, PDF, Notion, Confluence, anything not a text/markup file in this repo) — this skill is plain-text only.
- **Authoring from blank** — this skill reconciles existing material; generating a new doc from scratch is a different activity.
- **Code-level cleanup** — refactoring, formatting, import tidying live in other skills.
