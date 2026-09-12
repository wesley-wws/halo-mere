---
name: bump
description: Bump the plugin version across every entry in marketplace.json
argument-hint: "[major|minor|patch]"
---

# Version Bump

Bump the version number across all plugin metadata. All bundles in this repo share one version and are released together.

## Instructions

1. Read the current version from `.claude-plugin/marketplace.json` → `metadata.version`
2. Parse the version as `major.minor.patch`
3. Determine the bump type:
   - If the user provided an argument (`patch`, `minor`, or `major`), use it directly
   - If no argument was provided, review git changes since the last version bump (`git log`, `git diff`) and decide the appropriate level. Briefly explain your reasoning before applying
4. Apply the bump:
   - `patch`: increment patch (e.g., 0.4.1 → 0.4.2) for bug fixes, minor tweaks, wording changes
   - `minor`: increment minor, reset patch (e.g., 0.4.1 → 0.5.0) for new features, new skills/commands
   - `major`: increment major, reset minor and patch (e.g., 0.4.1 → 1.0.0) for breaking changes, major restructuring
5. Update every `version` field in `.claude-plugin/marketplace.json`:
   - `metadata.version`
   - `plugins[].version` for **every** entry (`halo-mere`, `halo-code`, `halo-think`, `halo-flow`), not just the first
   - There is no `.claude-plugin/plugin.json`; a root manifest would collide with the per-bundle `skills` arrays
6. Verify with `claude plugin validate .` and confirm no `version` string of the old value remains:
   `grep -c '"version": "{new}"' .claude-plugin/marketplace.json` should equal 5
7. Report the change: `Version bumped: {old} → {new}`
