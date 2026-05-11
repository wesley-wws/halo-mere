---
name: review-backend-arch
description: >
  Engineering-framework-level architecture review for .NET backend projects. Evaluates solution
  structure, module boundaries, domain partitioning, dependency direction, and long-term
  maintainability. Does NOT review implementation details.
  Manually invoked — not auto-triggered.
user-invocable: true
allowed-tools:
  - Read
  - Bash
  - Agent(Explore)
---

# Backend Architecture Review

Engineering-framework-level audit. Examines how the solution is partitioned into projects
and modules, whether domain boundaries are correctly drawn, whether module responsibilities
are clear, and whether the structure can sustain long-term iteration at 90+ quality.

**Scope:** Project/folder organization, module boundaries, dependency direction, domain
modeling, structural design decisions.

**Out of scope:** Individual class implementations, specific API usage patterns, code style,
algorithm correctness. Those belong in code review.

## Guiding Principles

**Occam's Razor** — Do not recommend structures, layers, or module splits the project does
not yet need. A flat structure that works is better than a "proper" layered architecture
that's half-empty. The burden of proof is on the new boundary. The review MUST include a
"Do Not Change" section.

**Dependency Inversion at the Module Level** — High-level modules (domain, application)
must not structurally depend on low-level modules (infrastructure, I/O). Check this in the
project reference graph — not in individual classes.

**Entropy Containment** — Complexity should be concentrated in well-bounded modules. When
a module becomes a grab-bag of unrelated responsibilities, the architecture has failed
regardless of how clean the code inside it is.

**Consistency as Load-Bearing Structure** — When two parts of the codebase organize the
same concern differently, every future contributor must guess which to follow. Architecture
drift is a decision-propagation failure.

**Don't Reinvent the Wheel** — If .NET or the ecosystem provides a structural solution
(middleware pipeline, Options pattern, built-in DI, standard project layouts), use it.
Custom architectural plumbing that duplicates framework capabilities is debt.

## Review Scope — Three Levels

1. **Solution level** — How are .csproj projects organized? What does the dependency DAG
   look like?
2. **Module level** — Within each project, how are folders/namespaces partitioned? Do they
   represent coherent domains?
3. **Boundary level** — What are the contracts between modules? Narrow and explicit, or
   wide and leaky?

Does NOT descend to individual class design or method implementations.

## Review Process

### Phase 1: Discovery

Map the structural terrain. Collect facts, do not judge.

Spawn an **Explore agent** to gather:

```
1. Project graph
   - All .csproj files and their <ProjectReference> links
   - Draw the dependency DAG

2. Module inventory
   - For each project: top-level folders (1-2 levels deep)
   - For each major folder: one-sentence summary of what it contains
   - Identify which folders represent domain/subdomain boundaries

3. Architecture intent
   - CLAUDE.md, docs/architecture.md, ADRs, or equivalent
   - Stated architectural style (Clean Architecture, Vertical Slice, Modular Monolith, etc.)
   - Any automated architecture enforcement (NetArchTest, ArchUnitNET in test .csproj)?
   - If no documentation exists, infer the intended style from the project graph shape

4. Structural metrics (counts, not implementations)
   - File count per project/folder
   - The 3-5 largest folders by file count
   - Any folders with < 3 files (possible over-partitioning)
```

**Before analysis, identify the architecture style.** Different styles have different
rules — a Vertical Slice project shouldn't be judged against Clean Architecture criteria.
If no clear style exists, that is itself a finding.

Output a structured fact sheet before proceeding.

### Phase 2: Analysis & Report

Evaluate each dimension. Read files only to understand what a module IS (its public
contracts, its folder organization) — not how it WORKS internally.

#### Scoring Dimensions

See `references/review-framework.md` for per-dimension guidance.

| # | Dimension | Key Question |
|---|-----------|-------------|
| 1 | Solution & Project Partitioning | Does each project represent a meaningful architectural unit? |
| 2 | Domain & Subdomain Boundaries | Are domains correctly identified with boundaries at natural seams? |
| 3 | Dependency Direction | Does the project graph enforce proper layering? |
| 4 | Module Responsibility | Can each module be described in one sentence without "and"? |
| 5 | Entropy Containment | Is complexity concentrated in the right modules? |
| 6 | Extensibility & Growth Path | Can new features/domains be added without restructuring? |
| 7 | Architecture Drift & Consistency | Does the structure follow a single coherent organizing principle? |

**Scoring scale:**
- 90-100: Sound — no structural action needed
- 75-89: Solid — minor improvements when convenient
- 60-74: Gaps — address before the next major feature
- 40-59: Significant — structural problems that compound
- 0-39: Blocking — architecture is actively hindering development

For dimensions below 90, identify:
- Which project(s) or module(s) are affected
- What principle is violated and why it matters for long-term iteration
- Concrete structural recommendation (move, split, merge, or add boundary)

### Report Template

**Match the user's language** — if the user writes in Chinese, the entire report is in Chinese.

```
# Backend Architecture Review

## Overall Assessment

| Dimension | Score | Summary |
|-----------|-------|---------|
| Solution & Project Partitioning | __/100 | ... |
| Domain & Subdomain Boundaries | __/100 | ... |
| Dependency Direction | __/100 | ... |
| Module Responsibility | __/100 | ... |
| Entropy Containment | __/100 | ... |
| Extensibility & Growth Path | __/100 | ... |
| Architecture Drift & Consistency | __/100 | ... |

**Overall Score: __ / 100**

---

## Issues

### Issue N: [Title]

**Severity:** Critical / Major / Minor
**Affected modules:** [Which projects/folders]
**What's wrong:** [Structural problem — not code-level]
**Why it matters:** [Impact on maintainability, extensibility, or team velocity]
**Recommended restructuring:** [Concrete module-level action]

---

## Do Not Change

Explicit list of structural decisions that are adequate. For each, name the tempting
"improvement" that should be resisted and why.

Not optional. A review that only lists problems drives over-architecture.

---

## Refactoring Roadmap

### Phase 1: [Theme] — Priority: Highest

| Structural Change | Rationale |
|-------------------|-----------|
| ... | Why this matters now |

Each phase must be independently shippable.

---

## Strategic Outlook

Think forward — this is the reviewer's independent judgment, not limited to what was asked.

1. **Pressure points** — Where will the architecture strain first as the project grows?
2. **Deferred decisions** — Structural decisions to resolve before the next major feature.
3. **What to NOT introduce** — Patterns/libraries/layers the team should resist.
4. **Architectural governance** — Are rules enforced mechanically (fitness functions,
   ArchUnitNET/NetArchTest in CI) or only by human review? If only human, recommend
   which rules to automate first.
5. **What else can be done** — Proactive structural improvements beyond the issues found.

---

## Architecture Guard Rails

Numbered conventions to prevent drift. Each rule enforceable in code review.

> N. **[Rule]** — [Cost of violation, one sentence.]
```
