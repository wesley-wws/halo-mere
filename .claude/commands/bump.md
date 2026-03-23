---
name: bump
description: Bump the plugin version in plugin.json and marketplace.json
argument-hint: "[major|minor|patch]"
---

# Version Bump

Bump the version number across all plugin metadata files.

## Instructions

1. Read the current version from `.claude-plugin/plugin.json`
2. Parse the version as `major.minor.patch`
3. Determine the bump type:
   - If the user provided an argument (`patch`, `minor`, or `major`), use it directly
   - If no argument was provided, review git changes since the last version bump (`git log`, `git diff`) and decide the appropriate level. Briefly explain your reasoning before applying
4. Apply the bump:
   - `patch`: increment patch (e.g., 0.4.1 → 0.4.2) — bug fixes, minor tweaks, wording changes
   - `minor`: increment minor, reset patch (e.g., 0.4.1 → 0.5.0) — new features, new skills/commands
   - `major`: increment major, reset minor and patch (e.g., 0.4.1 → 1.0.0) — breaking changes, major restructuring
5. Update the `version` field in these files:
   - `.claude-plugin/plugin.json` — `version`
   - `.claude-plugin/marketplace.json` — `metadata.version` and `plugins[0].version`
6. Report the change: `Version bumped: {old} → {new}`
