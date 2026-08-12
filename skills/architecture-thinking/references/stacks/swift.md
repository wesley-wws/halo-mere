# iOS / Swift Calibration Notes

When to read this file: after detecting `Package.swift`, `*.xcodeproj`, or `*.xcworkspace`.

These are **not** a pattern catalog. SwiftPM, MVVM, TCA, and the rest are already in the
model's training data. What follows are guard rails against the specific directions the model
over-flags, or misses, on this stack.

## Single-target is often correct

Many small iOS apps are single-target, and that is *correct*, not a finding. The bar for
"should this be split into SwiftPM packages?" is higher than for backends, because Xcode's
build-system overhead with many local packages is real and paid on every build. Recommend
splitting only when domain complexity, team boundaries, or build-time isolation justifies it.

## `public` everywhere is performative modularity

A "modular" Swift project where most types are `public` has no real module boundary:
refactoring is blocked by invisible call sites, which is exactly what a boundary is supposed to
prevent. Treat `public`-everywhere as effectively no boundary, regardless of what the SwiftPM
target graph declares.

## Singletons undo the declared graph

`SomeService.shared` reached from every module undoes whatever dependency direction the package
graph declares. Check for these *before* trusting any Lens 2 conclusion; the graph will look
clean while the runtime coupling is total.

## Storyboards spanning features

A `Main.storyboard` holding screens from many features is a modularization blocker. Call this
out when a project claims to be modular but ships one.
