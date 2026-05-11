---
name: architecture-thinking
description: >
  Activate when the user is reviewing the structure of an existing codebase, designing a
  new system or module layout, or debating a specific architectural decision (should this
  be split / merged / where this code belongs / where to draw a boundary). Trigger on
  English phrases like "review the architecture", "should I split this module", "where
  does this code belong", "I'm starting a new project — how should I structure it",
  "what's wrong with this layout", and on Chinese phrases like "评审架构", "架构评审",
  "整体怎么组织", "目录怎么搭", "这块该放哪", "新项目怎么搭目录", "该不该拆",
  "模块边界". Apply even when the user does not explicitly say "architecture" — any
  question about project / package / module organization, dependency direction, domain
  boundaries, or long-term structural soundness counts. A guiding skill, not a checklist:
  five principles, four lenses, and the disciplines of a senior architect. Includes
  per-stack calibration notes for .NET, Node/TypeScript (frontend and backend), and
  iOS/Swift; other stacks fall back to the universal lenses. Skill-style — shapes how the
  reply is structured and reasoned, replies inline; does not write files.
user-invocable: true
allowed-tools:
  - Read
  - Bash
  - Agent(Explore)
---

# Architectural Thinking

A way of thinking about software architecture — how projects, packages, modules, and
dependencies should be organized so the structure itself encodes intent and survives
long-term iteration. Principles, lenses, and disciplines made explicit, applied to
whatever codebase or design question is in front of you.

## What counts as "architecture" here

**In scope:** project / package / module organization; folder structure as it expresses
boundaries; the build-level dependency graph; placement of responsibilities; cross-cutting
concern strategy.

**Out of scope:** specific class designs, algorithm choices, API usage patterns, code
style, naming conventions inside a function. Those are implementation concerns — they go
in code review, not here.

The distinction matters: implementation problems can be fixed in one commit; architecture
problems compound with every feature added on top of them.

---

## Modes of Invocation

This skill activates in three contexts, each with a different output shape. Identify the
mode from the user's request **before** applying the lenses — the mode determines which
output template (at the end of this file) to produce. The principles, lenses, disciplines,
and stack calibration that follow this section are applied across all three modes; only
the input shape and the output template differ.

### Mode A — Reviewing an existing codebase

**Trigger:** The user asks to review, audit, or assess an existing system's structure —
"is this organized well?", "评审一下这个项目的架构", "what's wrong with this layout?"

**Flow:**

1. Detect the stack (markers listed under Stack Calibration Notes below).
2. Spawn an Explore agent to map the build manifests and their declared dependencies,
   top-level folders per project (1-2 levels deep), architecture intent documents
   (CLAUDE.md, AGENTS.md, ADRs, README), and rough file-count distribution.
3. Identify the intended style. If unclear, that itself is a finding.
4. Apply the four lenses. Form judgments as prose, not scores.
5. Produce the **Review Template**.

### Mode B — Guiding new design

**Trigger:** The user is starting fresh — "I'm building X, how should I structure it?",
"我要做一个 Y，怎么组织目录？", "新项目刚起步，先搭个架子"

**Flow:**

1. Identify the stack (if known) so the calibration notes below apply, and surface other
   constraints: actual scope, realistic growth trajectory, team size. If anything material
   is unknown, ask before recommending.
2. Apply Occam's Razor first: what's the *minimum* shape that works? Start there.
3. Identify the one or two future pressures that justify any additional structure.
4. Recommend a starting shape with explicit "don't add this yet" markers for structure
   that's tempting but premature.
5. Produce the **Design Template**.

### Mode C — Debating a specific decision

**Trigger:** The user has a concrete decision in front of them — "should I split this
module?", "where does this code belong?", "is this doing too much?", "这两个该不该合并？"

**Flow:**

1. Restate the decision precisely, including the stack context if known (the calibration
   notes below may change the answer).
2. Identify the 2-3 principles that bear most directly on this decision — not all five
   will apply to every choice.
3. Identify the *cost of being wrong* in each direction. Cost asymmetry usually points
   to the answer.
4. Recommend with rationale tied to the specific principles raised in step 2.
5. Produce the **Decision Template**.

### When a request straddles modes

Real conversations often blur the boundaries — "I'm reviewing X and wondering if we should
split Y", or "I'm adding a new feature, where does it go?" Pick the *dominant* mode and
produce its template, then add a short follow-up paragraph in the spirit of the secondary
mode. Don't force a hybrid template; don't refuse to engage one side of the question.

---

## The Five Principles

These are the bedrock. Every recommendation, every "Do Not Change" entry, every judgment
call should be traceable to one or more of these.

### Occam's Razor

Do not recommend structures, layers, or module splits the project does not yet need. A
flat structure that works is better than a "proper" layered architecture that's half-empty.
The burden of proof is on the new boundary, not on the absence of one.

This principle is why the output of an architectural review *must* include a "Do Not
Change" section. A review that only enumerates problems pushes the team toward over-
architecture — adding layers and abstractions whose cost is paid every day while their
benefit is hypothetical.

### Dependency Inversion at the Module Level

High-level modules — those expressing domain policy, business rules, application use cases
— must not structurally depend on low-level modules — those handling infrastructure, I/O,
framework adapters, platform SDKs. The check is on the *declared* project / package
reference graph, not on individual class imports. A manifest-declared dependency is the
boundary the team has agreed to defend; source imports can violate intent silently.

### Entropy Containment

Complexity is unavoidable, but it must live somewhere specific. A well-architected system
has a clear gradient — simplest at the policy / domain layer (where business invariants
live), growing toward the infrastructure / adapter layers (which deal with the messy real
world). When complexity spikes in the wrong place — business rules in an HTTP handler,
I/O calls in a domain entity — the architecture has failed, however clean the code is.

Equally: when a module becomes a grab-bag of unrelated responsibilities ("Common", "Utils",
"Shared", "Helpers"), the architecture has failed by accumulation.

### Consistency as Load-Bearing Structure

When two parts of the codebase organize the same concern differently, every future
contributor must guess which to follow. Architecture drift is a decision-propagation
failure: the original decision didn't transmit to the next contributor, who made their
own. Consistency is what makes a structure *teachable* — and an unteachable structure
deteriorates with every new hand.

This is why mixed patterns (Next.js `pages/` and `app/` routers coexisting; vertical
slices and traditional services in the same .NET project; UIKit and SwiftUI without a
stated direction) deserve calling out even when each is locally fine.

### Don't Reinvent the Wheel

If the framework or platform provides a structural solution — DI containers, middleware
pipelines, the framework's recommended folder layout, the package manager's workspace
features, the platform's module system — use it. Custom architectural plumbing that
duplicates framework capabilities is debt: it has to be learned, maintained, and debugged
by everyone, and it usually does the same job worse.

---

## The Four Lenses

These are the questions an architect asks. Apply them in any order. They are mental moves
to make, not a checklist to fill in. Findings come from applying them — not from filling a
scorecard.

### Lens 1: Does the structure reveal intent?

Walk the project graph and folder layout *as if you'd never seen the code*. Can you tell
from the structure alone what the system is trying to be? A clean inward-pointing DAG
with named domain modules tells a story — "this is a Clean Architecture system around
these three domains." A flat web of mutual dependencies tells a different story — "this
grew organically; no one was steering."

The structure should be a teaching tool for the next contributor. If understanding the
project layout requires opening every file, the structure has stopped working as
architecture.

Look for:
- Top-level folders / projects named by domain (good) or by technical role (`Services/`,
  `Repositories/`, `Components/`, `Utils/`) — the latter is the classic sign of missing
  domain modeling.
- All subdomains follow the same organization, or each its own world?
- What docs say matches what the code does?
- The same concept named differently across modules ("User" vs "Buyer" vs "Account") —
  often signals distinct bounded contexts that deserve separate modules.

### Lens 2: Does the dependency graph point the right way?

The build-level graph — `<ProjectReference>` in .NET, `targets:`/`dependencies:` in
`Package.swift`, `dependencies` and `workspaces` in `package.json`, `references` in
`tsconfig.json` — is the authoritative record of how layers were *intended* to depend on
each other. Read it before reading any code.

In a well-architected system, arrows point inward: Infrastructure → Application → Domain;
Adapters → Core; Feature → Domain → Kernel. Domain / Core depends on nothing but base
libraries.

The graph is also where infrastructure leakage shows up: an ORM package referenced by a
Domain project, a UI framework referenced by business logic, an HTTP client in Core. Even
if no class uses these, the reference itself contaminates the layer.

Look for:
- Inward-pointing DAG? Or flat / circular / shortcut-laden?
- Domain / Core declaring infrastructure dependencies?
- Application reaching into Infrastructure directly (it shouldn't — the host /
  composition root bridges via DI).
- A "shortcut" reference made "just for one thing" — these accumulate.

### Lens 3: Does complexity live where it belongs?

Look at the size and shape of each module. Where is the bulk? A 200-file module in a
1000-file system might be legitimate (inherent complexity in that domain) or might be a
god module that absorbed everything. The way to tell is to look at *what* is in it, not
just *how much*.

Cross-cutting concerns (auth, logging, error handling, validation, telemetry) deserve
particular attention: are they solved architecturally — one middleware, one interceptor,
one DI registration — or has each module solved them separately?

Look for:
- Modules whose responsibility you can't state in one sentence without "and".
- Catch-all names: "Common", "Shared", "Utils", "Helpers", "Misc" — at the workspace root
  these can be legitimate if narrow; inside a domain they almost always indicate
  misplacement.
- Cross-cutting concerns reinvented per module.
- State ownership unclear — multiple modules reading/writing the same store, the same DB
  tables, or the same singletons.

### Lens 4: How will it absorb growth?

The architecture isn't proven by today's feature set; it's proven by the next ten
features, the next two integrations, the next subdomain. Run the mental simulation:

- "If we add a new subdomain next month, where does it go? What changes?"
- "If we swap the database / message bus / auth provider, how many modules are affected?"
- "If a new team starts contributing, can they learn the structure from one folder?"

A good architecture has *one* answer to "add a feature": add a new module with one new
edge in the graph. A failing architecture requires editing five existing modules to add
anything new.

Guard against the opposite failure too. An architecture *over-prepared* for growth that
won't happen — elaborate extension points, plugin systems, abstraction layers — is its
own debt. Match the prepared-for-growth to the project's actual trajectory, not to a
generic "what if it scales" anxiety.

---

## The Disciplines

The principles tell you *what* to think; the disciplines describe *how* to think.

**Read the build graph before reading any code.** The `.csproj` references, the
`Package.swift` targets, the `package.json` workspaces — these are the architecture's
declared shape. Code that contradicts them is implementation drift; code that respects
them confirms intent. Reading code first gives a noisy view; reading the graph first
gives the team's stated intent, which is what's being evaluated against.

**Identify the intended style before judging anything.** A Vertical Slice .NET project
looks "wrong" if judged against Clean Architecture rules, and vice versa. Feature-Sliced
Design has its own logic that Atomic Design doesn't share. Before calling anything a
problem, identify what the team was trying to build — from documentation if present,
from the graph shape if not — and apply the right standard. If no coherent style is
identifiable, *that itself is the finding*, not "they should have done X."

**Distinguish architecture from implementation.** If a finding could be addressed by
editing one file, it's almost certainly an implementation issue. Architecture findings
affect multiple modules, change the dependency graph, or require restructuring rather
than refactoring. Implementation findings belong in code review.

**Resist new boundaries.** Every new module, new layer, new abstraction is a tax paid
daily by everyone navigating the system. The burden of proof is on the new boundary,
always. This is Occam's Razor operationalized: in a review, the "Do Not Change" section
is where this discipline lives — explicit defense of decisions that *could* be
"improved" but shouldn't be.

**Match the user's language.** If the user writes in Chinese, the report is in Chinese.
If they use specific vocabulary (DDD bounded contexts, vertical slices, hexagonal ports,
SwiftUI navigation, feature-sliced layers), use the same vocabulary back. Mirroring is
part of making the output usable in their actual workflow.

---

## Stack Calibration Notes

The principles, lenses, and disciplines above are stack-agnostic. But the model's default
inclinations on certain dimensions need calibration per stack, or it will over-flag in
predictable directions. The notes below are *not* a pattern catalog — those facts are in
the model's training data already. They are guard rails against specific default biases.

Detect the stack from these markers at the repo root:

| Repo marker | Stack |
|-------------|-------|
| `*.sln` / `*.csproj` | .NET |
| `package.json` with web framework deps (react, vue, angular, svelte, next, nuxt) | JS/TS frontend |
| `package.json` with server framework deps (express, nestjs, fastify, koa, hono) | Node/TS backend |
| `Package.swift` / `*.xcodeproj` / `*.xcworkspace` | iOS / Swift |

For stacks not listed (JVM, Android, Flutter, Go, Rust, Python, etc.), apply the principles
and lenses directly and note in the output that no stack-specific calibration was applied.

**Project-specific facts belong in the project, not in this skill.** Statements like "we
use Clean Architecture with X/Y/Z projects", "our Common module is intentionally narrow",
"we chose against MediatR" — these go in the project's `CLAUDE.md` / `AGENTS.md`. Read
them as part of the discovery step in any mode. This skill brings the thinking framework;
the project brings the project facts.

### .NET (`.csproj` solutions)

- Most medium-sized .NET backends need only 3-4 source projects. Many .NET solutions are
  *over-partitioned* because tutorials show 5-7 projects. Recommend consolidation when
  projects always change together; do not recommend further splits without strong cause.
- Cross-check `<PackageReference>` alongside `<ProjectReference>` — infrastructure
  packages (EF Core, ASP.NET Core, message-bus libraries) declared in the wrong project
  are layer pollution even if no class uses them. The reference itself contaminates.
- Mixed MediatR vertical-slice and traditional-service patterns in one project without a
  stated migration direction is a drift signal worth calling out specifically.

### JS/TS (`package.json` — frontend and Node backend)

- JS/TS is the stack where "module boundary" most often means *nothing enforced at all*.
  Without TypeScript project references or `workspaces`-declared package boundaries,
  dependency direction is convention-only. Flag this gap explicitly rather than
  concluding Lens 2 is satisfied just because imports happen to point the right way
  today.
- ORM-generated types (Prisma, Drizzle) used as domain entities is the JS/TS equivalent
  of EF Core in Domain — couples domain to schema and migration cadence. Apply Lens 2
  the same way as for .NET.
- Mixed framework conventions (Next.js `pages/` and `app/` coexisting; multiple state-
  management libraries without a stated strategy) are common drift findings — call them
  out under Lens 1 even when each pattern is locally fine.

### iOS / Swift

- Many small iOS apps are single-target — that is *correct*, not a finding. The bar for
  "should this be split into SwiftPM packages?" is higher than for backends because
  Xcode build-system overhead with many local packages is real. Recommend splitting only
  when domain complexity, team boundaries, or build-time isolation justifies it.
- A "modular" Swift project where most types are `public` has *performative* modularity
  — refactoring is blocked by invisible call sites. Treat `public`-everywhere as
  effectively no module boundary, regardless of the declared SwiftPM target graph.
- Singleton service locators (`SomeService.shared`) reached from every module undo
  whatever dependency direction the package graph declares. Check for these before
  trusting any Lens 2 conclusion.
- Storyboards spanning multiple features are a modularization blocker — call this out
  when a project claims to be modular but ships a `Main.storyboard` with screens from
  many features.

---

## Output Templates

This skill shapes how a reply is *structured and reasoned* — it does not produce file
artifacts. Output goes inline in the conversation reply. Templates frame the output; the
*content* comes from applying the principles and lenses. Match the user's language.

If the user later asks for the output as a saved file, that's a separate action — do it
then. Don't pre-emptively write files; this is a thinking skill, not a command.

### Review Template (Mode A)

```
# Architecture Review — [Stack]

## What this system is trying to be

[One paragraph. The intended architectural style as read from the structure and any
documentation. If no clear intent is visible, say so explicitly — it's a finding.]

## What's working

[Prose. Where the structure successfully encodes intent, contains complexity, and sets
up future iteration. Be specific. Reviews that only enumerate problems push teams toward
over-architecture.]

## What isn't

### [Issue title]
**Severity:** Blocking / Major / Minor / Drift
**Affected:** [Which projects / modules / folders]
**The problem:** [Architecture-level — not code-level]
**Why it compounds:** [What this will cost over the next N features / months]
**Structural fix:** [Concrete module-level action]

[Repeat per issue, ordered by severity.]

## Do Not Change

[Required. Explicit list of structural decisions that are adequate and should be defended
against well-intentioned "improvements." For each, name the tempting change and why it
would be wrong. Not optional — a review without this section drives over-architecture.]

## Pressure points ahead

[The 1-3 places the architecture will strain first as the project grows. Tie each to a
specific likely future change.]

## Architecture Guard Rails

[Numbered conventions to prevent the drift identified above. Each phrased as a rule a
team member could check against in code review.]
```

### Design Template (Mode B)

```
# Architectural Guidance — [System / module name]

## Constraints I'm designing against

[Restate scope, growth trajectory, team size, stack as understood. Flag any unknowns.]

## The minimum shape

[The simplest structure that meets the stated requirements. Justified per Occam.]

## Future pressures that shape this

[1-3 realistic future pressures that justify any structure beyond the minimum. Generic
"what if it scales" anxiety doesn't count.]

## What to add now

[Concrete recommendations beyond the minimum, each tied to a specific future pressure.]

## What NOT to add yet

[Tempting additions to defer until a real pressure justifies them. This is where Occam's
Razor does most of its work in greenfield design.]

## Anchors

[The 2-4 architectural decisions that, once made, should be defended even as the system
grows. These are the structure's load-bearing claims.]
```

### Decision Template (Mode C)

```
# Architectural Decision — [Restate]

## What the relevant principles say

[List the 2-3 principles that bear most directly on this decision, one sentence each on
what they imply. Omit principles that don't apply — not every decision touches all five.
Naming a principle and writing a filler sentence is worse than not naming it.]

- **[Principle name]:** [Implication for this specific decision]
- **[Principle name]:** [Implication]

## Cost of being wrong each way

- **If we do X and we shouldn't have:** [Concrete cost]
- **If we don't do X and we should have:** [Concrete cost]

## Recommendation

[The judgment, tied to the principle(s) it rests on and the cost asymmetry that makes it
the right call.]
```
