---
name: code-decomposition
description: A thinking framework for breaking complex code into focused, composable units — applies fractally from a three-line function to a 500-line class to an entire module. Use whenever the user asks to refactor, restructure, split, extract, decompose, simplify, or review code for complexity — e.g. "this class is doing too much", "god class", "long method", "extract a helper", "break this up". Skill-style — shapes how the reply is structured and reasoned, replies inline; does not write files.
---

# Code Decomposition Guide

A thinking framework for breaking complex code into focused, composable units. The goal is not to follow rules mechanically but to reason about where the natural seams in your code are — and cut along them.

## The Core Question

The Single Responsibility Principle boils down to one question: **"What would cause this unit to change?"**

If the answer lists multiple independent reasons — say, a change in business rules AND a change in persistence format — the unit has multiple responsibilities and is a candidate for splitting. This question applies fractally: to a three-line function, a 500-line class, or an entire module.

A useful companion is the **newspaper test**: can you describe what this unit does in a single sentence without using "and"? If not, it likely does more than one thing.

SRP is not about size. A 300-line class that manages a tightly coupled state machine may have exactly one responsibility. A 30-line class that fetches data and formats it for display has two.

## Modes of Invocation

The skill applies in three modes. Identify which mode the user is in before responding — the output shape differs.

- **Mode A — Review**: the user shows existing code and asks where it could be decomposed. Output: candidate seams with rationale and priority, not a full rewrite.
- **Mode B — Execute**: the user has already decided to decompose and wants help doing it. Output: follow the Decomposition Workflow below.
- **Mode C — Debate**: the user names a specific extraction and asks whether it is worth it. Output: a verdict (split / don't split / it depends) with one or two sentences grounded in cohesion, coupling, or the "When NOT to Split" rules.

If the user's request straddles modes (e.g., "look at this and tell me what to do"), do Mode A first and ask before moving into Mode B. The wrong seam is expensive to undo once code has been moved.

## The Tradeoff Frame

Decomposition is fundamentally a tradeoff: splitting improves cohesion within each piece but increases coupling between them. **A good split is one where the cohesion gains clearly outweigh the coupling costs.** Splits that look like progress but force callers to know more, share mutable state across the new boundary, or require constant back-and-forth communication are not decomposition — they are redistribution of complexity. Look for clusters of fields and methods that interact with each other but rarely with the rest; those clusters are the candidates.

## Decomposition Workflow

When you decide to decompose (Mode B), follow this sequence. Do not skip steps — the explicit pass is what catches the wrong seam early.

1. **Identify** — List what the unit does. Write each responsibility as a short phrase. If the list has more than one item, decomposition may help.
2. **Cluster** — Group related responsibilities. Fields and methods that reference each other form natural clusters. Each cluster is a candidate for extraction.
3. **Name** — Give each cluster a name. If you cannot find a clear noun or verb-phrase, the cluster may not represent a real concept — reconsider the grouping.
4. **Define boundaries** — For each new unit, decide its public interface. What does the outside world need from it? Everything else stays internal.
5. **Extract** — Move code into the new units. The original unit often becomes a thin coordinator that delegates to the extracted pieces rather than doing work itself.
6. **Verify** — Confirm each new unit passes the newspaper test (one-sentence purpose), coupling between new units goes through interfaces, and existing tests still pass.

## Disciplines

Disciplines that separate a careful decomposition from a destructive one. The standard refactoring patterns (Extract Method, Extract Type, Extract Interface, Move Method) are assumed knowledge — these disciplines govern *when and how* to reach for them.

- **Read the whole unit before proposing seams.** Proposing splits after skimming half a class produces seams that fall apart on the parts you didn't read. The cost of a wrong cut is far higher than the cost of reading.
- **If you cannot name the cluster, do not extract it.** A `Helper`, `Manager`, `Util`, or `Processor` name is a confession that the cluster is not a real concept. Renaming is not optional — it's the test that the cluster exists.
- **Distinguish complex-single-responsibility from multi-responsibility.** A long state machine, a parser with many branches, or a hot path with tight invariants is often one thing done thoroughly. Size alone is not evidence of multiple responsibilities.
- **Length is not a decomposition trigger; lack of cohesion is.** Before extracting a new type at all, reach for the lightest language-native grouping (e.g. Swift `extension` + `// MARK:`, C# `partial class` / `region`, a module-level helper function, `internal` vs `private` visibility). Then prefer the lightest pattern: Extract Method before Extract Type; Extract Type before Extract Interface; Extract Interface only when a second implementation or a test substitute is actually needed.
- **Match the scope of the user's request.** If the user asks about one method, do not propose a module rewrite. Surface the larger opportunity in one sentence and let them invite the bigger discussion.
- **Match the user's language.** If the user is writing in Chinese, reply in Chinese. The skill is content, not language.

## When NOT to Split

Knowing when to stop is as important as knowing when to start.

- **Premature abstraction.** If there is only one implementation and no foreseeable variation, extracting an interface adds indirection without benefit. Wait until a second use case emerges.
- **Coherent state machines.** When state transitions depend on multiple pieces of data changing together atomically, splitting that state across units introduces synchronization complexity. A class managing tightly coupled state is not a god class — it is doing one complex thing.
- **Split increases coupling.** If the extracted pieces need constant back-and-forth communication or share mutable state, the "decomposition" just redistributed complexity. Revert and look for a different seam.
- **Fragments without standalone meaning.** If an extracted unit has no purpose outside its parent context, it is not a real abstraction. Prefer a private helper method or nested type.

## Stack-Specific Anti-Biases

The universal principles above cover most stacks. This section only lists biases that the principles alone don't catch — places where a common default looks like decomposition but isn't, or where the natural seam is non-obvious in a way the language doesn't telegraph.

- **JS/TS — barrel files are not decomposition.** A `index.ts` that re-exports the contents of a folder hides module boundaries, frequently produces circular imports, and makes the "unit" ambiguous. If a folder needs a barrel to feel decomposed, the decomposition isn't real. Treat barrel files as suspect when reviewing.
- **JS/TS (Node) — a long coordinator function is not a god function.** A function that orchestrates other functions reflects the number of steps in the workflow, not the number of responsibilities it owns. Resist extracting "service" / "manager" layers inside a module just because the top-level function is long — the module itself is already the unit.

For other stacks, fall back to the universal disciplines — particularly *"reach for the lightest language-native grouping before extracting a new type."*

## Output Templates

Shapes for how the reply should be structured. These are skill-style templates — replied inline, not written to files.

### Mode A — Review

```
## Decomposition Review: <unit name>

### Current responsibilities
- <one-line phrase per responsibility>

### Candidate seams
1. **<name>** — <what cluster> · <pattern> · <risk: low/med/high>
2. ...

### Do Not Change
- <parts that look complex but are coherent — explain why>

### Suggested order
<which seam to tackle first and why>
```

### Mode B — Execute

Follow the six-step Decomposition Workflow above. Show the result of each step.

### Mode C — Debate

```
**Verdict:** <split / don't split / it depends on X>

**Reasoning:** <one or two sentences grounded in cohesion, coupling, or a "When NOT to Split" rule — name which one>
```

Naming a heuristic and writing a filler sentence is worse than not naming it. If the verdict is "it depends", say what it depends on in concrete terms — what evidence would flip the decision.
