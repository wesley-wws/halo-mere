# Output Templates

When to read this file: once the mode is identified (A, B, or C), before writing the reply.
Read only the template for the mode in play.

This skill shapes how a reply is *structured and reasoned* — it does not produce file
artifacts. Output goes inline in the conversation reply. Templates frame the output; the
*content* comes from applying the principles and lenses. Match the user's language.

If the user later asks for the output as a saved file, that's a separate action — do it
then. Don't pre-emptively write files; this is a thinking skill, not a command.

## Review Template (Mode A)

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

## Design Template (Mode B)

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

## Decision Template (Mode C)

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
