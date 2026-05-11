---
name: code-decomposition
description: Guide for splitting large or complex classes, methods, and modules into smaller, focused units. Use this skill whenever refactoring, restructuring, or reviewing code for complexity — when a file, class, or function feels too large, has too many responsibilities, or is hard to understand. Also consult when discussing architecture boundaries, module organization, cohesion, coupling, or when the user mentions "god class", "long method", "extract", "split", "break up", or "decompose". Applies to any language with optional Swift-specific guidance.
---

# Code Decomposition Guide

A thinking framework for breaking complex code into focused, composable units. The goal is not to follow rules mechanically but to reason about where the natural seams in your code are — and cut along them.

## The Core Question

The Single Responsibility Principle boils down to one question: **"What would cause this unit to change?"**

If the answer lists multiple independent reasons — say, a change in business rules AND a change in persistence format — the unit has multiple responsibilities and is a candidate for splitting. This question applies fractally: to a three-line function, a 500-line class, or an entire module.

A useful companion is the newspaper test: can you describe what this unit does in a single sentence without using "and"? If not, it likely does more than one thing.

SRP is not about size. A 300-line class that manages a tightly coupled state machine may have exactly one responsibility. A 30-line class that fetches data and formats it for display has two.

## Cohesion, Coupling, and Boundaries

Decomposition is fundamentally a tradeoff: splitting improves cohesion within each piece but can increase coupling between them. A good split is one where the cohesion gains clearly outweigh the coupling costs. When you spot clusters of fields and methods that interact with each other but rarely with the rest of the class, those clusters are natural extraction candidates.

In practice, clean boundaries follow from low coupling:

- **Communicate through interfaces, not internals.** Callers depend on what a unit promises, not how it works inside.
- **Minimize the interface surface.** Each public member is a commitment — fewer promises mean more freedom to evolve internally.
- **Changes inside should not force changes outside.** If splitting means every caller must be rewritten, the boundary is in the wrong place.
- **Dependencies flow one direction.** Circular dependencies between extracted units indicate the split is wrong or a shared abstraction is missing.

## Extraction Patterns

Ordered from lightest to heaviest — prefer the lightest pattern that solves the problem:

| Pattern | When to reach for it |
|---|---|
| **Extract Method** | A block has a describable intent distinct from its surroundings. Lowest risk, try first. |
| **Extract Type** | A cluster of state changes together and can be named as a noun. |
| **Extract Interface** | Consumers depend on behavior, not a specific implementation; or you need test substitutes. |
| **Move Method** | A method primarily operates on another type's state. |

## Decomposition Workflow

When you decide to decompose, follow this sequence:

1. **Identify** — List what the unit does. Write each responsibility as a short phrase. If the list has more than one item, decomposition may help.
2. **Cluster** — Group related responsibilities. Fields and methods that reference each other form natural clusters. Each cluster is a candidate for extraction.
3. **Name** — Give each cluster a name. If you cannot find a clear noun or verb-phrase, the cluster may not represent a real concept — reconsider the grouping.
4. **Define boundaries** — For each new unit, decide its public interface. What does the outside world need from it? Everything else stays internal.
5. **Extract** — Move code into the new units. The original unit often becomes a thin coordinator that delegates to the extracted pieces rather than doing work itself.
6. **Verify** — Confirm each new unit passes the newspaper test (one-sentence purpose), coupling between new units goes through interfaces, and existing tests still pass.

## When NOT to Split

Knowing when to stop is as important as knowing when to start.

- **Premature abstraction.** If there is only one implementation and no foreseeable variation, extracting an interface adds indirection without benefit. Wait until a second use case emerges.
- **Coherent state machines.** When state transitions depend on multiple pieces of data changing together atomically, splitting that state across units introduces synchronization complexity. A class managing tightly coupled state is not a god class — it is doing one complex thing.
- **Split increases coupling.** If the extracted pieces need constant back-and-forth communication or share mutable state, the "decomposition" just redistributed complexity. Revert and look for a different seam.
- **Fragments without standalone meaning.** If an extracted unit has no purpose outside its parent context, it is not a real abstraction. Prefer a private helper method or nested type.

## Swift-Specific Notes

Where Swift's features change the decomposition approach:

- **Protocol-Oriented Composition.** Conform a type to multiple focused protocols rather than building a deep class hierarchy. Protocol extensions provide shared default implementations without subclassing — a lighter alternative to inheritance for shared behavior.
- **Value vs. Reference semantics.** Use structs and enums for extracted state containers (data that is copied, not shared). Use classes or actors for units with identity or lifecycle. Actors additionally provide a concurrency boundary, making them natural choices when the extracted unit manages shared mutable state.
- **Extensions for lightweight grouping.** Swift extensions can group related methods on the same type without extracting a new type. This is a softer alternative to Extract Type when the methods share the same state but serve different concerns — organize with `// MARK:` sections before deciding a full extraction is warranted.
