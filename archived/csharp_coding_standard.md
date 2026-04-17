# C# Coding Standard

## Naming

| Target | Convention |
|--------|-----------|
| Class / Struct / Constant / Enum / Method / Property / Namespace / Project | PascalCase |
| Interface | PascalCase, prefix `I` |
| Generic type param | `T` or `K` |
| Field | `_camelCase`, private only |
| Variable / Parameter | camelCase |

- Enum values: PascalCase
- Methods: verbs or verb-object pairs
- Properties: no `Get`/`Set` prefix
- Files/Projects: match assembly and namespace
- No Hungarian notation; no redundant prefixes/suffixes
- Append design-pattern name as suffix where applicable (e.g., `OrderFactory`, `PaymentAdapter`)
- Booleans: prefix `Can`, `Is`, or `Has`
- Unit tests: `MethodName_StateUnderTest_ExpectedBehavior`

## Code Style

- Composition over inheritance
- File-scoped namespace; `using` directives at top; one class per file
- Always use explicit braces
- Separate logical groups with blank lines
- Consistent parameter order across overloads
- Return empty collections, not null
- Avoid boxing/unboxing; use null-conditional operators; prefer `as` + null check
- Use `Span<T>` / `Memory<T>` for performance-sensitive paths
- Consider object pooling for frequently allocated objects
- Avoid deep method-call chaining

## Flow Control

- No recursion
- No method calls inside conditionals
- Ternary only for simple conditions
- No `== true` / `== false` comparisons
- Extract shared code out of if/else branches

## Exception Handling

- No exceptions for flow control
- Re-throw with `throw`, never `throw ex` (preserves stack)
- No empty catch blocks
- Set `innerException` when wrapping exceptions
- Use the correct exception type; never `NotImplementedException` in production

## Comments

- Avoid obvious comments; prefer self-documenting code
- Document complex logic; use task-list flags: `TODO`, `UNDONE`, `Hack`
- XML docs: cover all params, return values, and exceptions; use `<example>` and `<remarks>` where helpful

## Class Member Order

1. Constants / static readonly fields
2. Fields
3. Properties
4. Constructors / Destructors
5. Methods
6. Nested types
7. Static members

Within each group: `private` → `internal` → `protected` → `public`
