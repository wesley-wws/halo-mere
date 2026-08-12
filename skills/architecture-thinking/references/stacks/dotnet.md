# .NET Calibration Notes

When to read this file: after detecting `*.sln` / `*.csproj` at the repo root, before forming
any judgment.

These are **not** a pattern catalog. Clean Architecture, Vertical Slice, MediatR, and the rest
are already in the model's training data. What follows are guard rails against the specific
directions the model over-flags on this stack.

## Resist further partitioning

Most medium-sized .NET backends need only 3-4 source projects. Many .NET solutions are
*over-partitioned* because tutorials show 5-7 projects. Recommend consolidation when projects
always change together; do not recommend further splits without strong cause.

This inverts the usual review instinct. On a .NET solution, "add a project" is more often the
wrong answer than "merge two projects."

## Read `<PackageReference>`, not just `<ProjectReference>`

Infrastructure packages (EF Core, ASP.NET Core, message-bus libraries) declared in the wrong
project are layer pollution even when no class uses them. The reference itself contaminates:
it is a standing invitation, and the next contributor will accept it.

## Mixed patterns in one project

MediatR vertical slices coexisting with traditional service classes, with no stated migration
direction, is a drift signal worth calling out specifically. Each pattern is locally fine;
the absence of a stated direction is the finding (see the Consistency principle).
