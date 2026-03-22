---
name: structured-output
description: Provides principles and formatting rules for structured, precise written output. Most valuable when the output spans multiple sections, will be referenced or reused, or will be consumed by AI downstream. Not intended for conversational replies or single-sentence answers.
---

# Structured Output

## Principles

Before structuring, identify the output type and the reader's goal — this determines which structural patterns apply. Ordered by scope: structural principles (1–4) govern how content is organized; precision principles (5–8) govern how language is used.

### 1. Lead with What Matters Most

Put the most important information first. What that means depends on context:

- **Analytical / decision**: open with the conclusion or recommendation
- **Instructional / tutorial**: open with what this enables, or the first required action
- **Reference / lookup**: surface the most frequently needed item first

The underlying rule: don't make the reader dig for the key information.

### 2. Structure Signals Purpose

The organization should immediately tell the reader what they'll get. How headings work depends on the output type:

- **Analytical**: headings state conclusions — `## Revenue declined 12% in Q3`, not `## Revenue`
- **Instructional / reference**: headings are precise navigation labels — `## Installation`, not `## Getting the software set up`

Either way, a heading should answer "what will I get from reading this section?" If it can't, the section lacks a clear purpose.

### 3. MECE Grouping
Organize into 3–5 categories that are mutually exclusive and collectively exhaustive. Overlap signals a structural flaw; gaps signal incomplete analysis.

### 4. Logical Ordering
Choose one ordering principle for sibling sections and apply it consistently:
- **Temporal**: phase 1 → phase 2 → phase 3
- **By importance**: critical → significant → minor
- **Causal**: cause → effect → response
- **Sequential**: input → process → output

### 5. Terminology Consistency
Use one term per concept throughout. Synonyms create ambiguity for both human and AI readers. Define a term once at first use if clarification is needed.

### 6. Explicit over Implicit
- Avoid dangling pronouns: "it", "this", "the above" — name the referent directly
- State assumptions, constraints, and scope; don't leave them to be inferred
- Mark what is undecided: `TBD: owner for phase 2`

### 7. Separate Information Types
Clearly distinguish:
- **Fact**: what is currently true
- **Decision**: what has been agreed
- **Recommendation**: what is suggested but not yet decided
- **Open question**: what still needs resolution

### 8. Specific over General
Use concrete data, names, and measurements instead of qualifiers. Specific language is simultaneously more precise and more concise — it eliminates the need for hedging or disambiguation.

- Weak: "significantly improved response time"
- Strong: "P99 latency dropped from 200ms to 120ms"

Corollary: one claim per sentence. Two assertions in one sentence dilute both.

## Formatting Rules

**Headings**: `#` / `##` / `###` hierarchy. No more than 3 levels in most documents.

**Bold**: Only for critical terms or decisions at first occurrence. Overuse makes bold meaningless.

**Tables**: Use when comparing 3+ items across 2+ dimensions. Higher information density, lower token cost than prose.

**Lists**: For parallel, enumerable items without connecting logic. If bullets need "however" or "therefore" to flow, use prose instead.

**Key-value pairs**: Prefer `Owner: Wesley` over "Wesley is responsible for this." Shorter and unambiguous.

**Length**: Match length to complexity. Simple answers don't need headings or sections.

## Anti-Patterns

Remove before finishing:

- **Redundancy**: same content appearing in multiple places
- **Filler conclusions**: "In summary", "To conclude" — structure already signals this
- **Hedging noise**: "It's worth noting that", "Importantly", "It should be mentioned that"
- **Dangling references**: "As mentioned above", "See the previous section"
- **Preamble**: "This document aims to..." — start with the content directly
- **Closing pleasantries**: "I hope this helps", "Feel free to ask follow-up questions"

