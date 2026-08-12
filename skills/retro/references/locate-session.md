# Locating the Session File and Picking the Segment

## How the session directory path is derived

1. **Config root**: typically `~/.claude` (standard install) or `~/.claude-console` (some distributions). If the global CLAUDE.md explicitly specifies one (e.g. `Your config directory is ~/.claude-console`), use that.
2. **Session directory**: `<config-root>/projects/<encoded-cwd>/`

   Encoding rule from current `pwd`:
   - drive-letter colon (`:`) → `--`
   - path separators (`\` or `/`) → `-`

   Examples:
   - cwd = `/home/user/proj/my-app` → `home-user-proj-my-app`
   - cwd = `D:\Code\MyApp` → `D--Code-MyApp`

Each session file is `<session-uuid>.jsonl`. Sub-agent threads live in `subagents/agent-*.jsonl` — skip these, they are not main sessions.

## Locating the right file per mode

### Default (`/retro`, no args)

1. Prefer environment variable `$CLAUDE_SESSION_ID` to construct the file path directly.
2. Otherwise `ls -t *.jsonl | head -5`, then `head -1` each candidate and compare its `cwd` field; the first match is the current session.
3. If neither works, tell the user: "Cannot identify the current session — please use `/retro --last` or `/retro <id>`", then stop.

### `--last`

`ls -t *.jsonl | head -5`, exclude the current sessionId, pick the first.

### `<session-id>`

Read `<session-id>.jsonl` directly. Error and return if missing.

## Segment detection (applies to `/retro` default mode only)

Users often `/clear` between unrelated tasks within one session. `/retro` automatically picks the most recent segment worth analyzing. `--full`, `--last`, and `<session-id>` modes skip segment detection entirely.

1. **Find clear boundaries**: `grep -n 'command-name>/clear<' <file>` — each match is a segment boundary. Total segments = matches + 1.
2. **Pick the best segment**:
   - Start with the **last segment** (after the last `/clear`).
   - If that segment is too short (< 5 user messages OR < 100 jsonl lines) — the user probably just cleared — **silently fall back** to the previous segment (between the second-to-last clear and the last clear; or from file start to first clear if only one clear exists).
   - If no clears found, analyze the entire file.
   - If all segments are too short, tell the user: "No segment has enough content to retro — try `/retro --full`."
3. **Add a scope header line to the report**: `[segment N/M — from <clear-timestamp> to <end-timestamp-or-"end">]` where M = total segments (clear count + 1), N = which segment is being analyzed. Extract the timestamp from the `/clear` event's `"timestamp"` field. This tells the user exactly what was analyzed — especially important when auto-fallback selected a non-obvious segment. See `report-template.md` for scope-header forms in the other modes.

## Format-assumption caveat

This skill assumes the Claude Code jsonl format as observed in late 2025 / early 2026. Specifically:

- `/clear` events appear as `<command-name>/clear</command-name>` inside a content line
- The first line of each session file carries `sessionId`, `cwd`, and `gitBranch` fields
- Each event carries a `"timestamp"` ISO field
- `"is_error":true` marks a failed tool response
- Sub-agent threads live in `subagents/agent-*.jsonl` under the same session dir (skip these)

If a grep returns no matches where matches are expected, **inspect a few lines with `head` to detect format drift, then tell the user** rather than silently continuing on bad assumptions. Claude Code's transcript schema is not a published contract; format drift is a real risk and should fail loud, not silent.
