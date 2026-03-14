# S01: Welcome message in session header — Research

**Date:** 2026-03-14

## Summary

The `session_start` hook in `src/resources/extensions/gsd/index.ts` (lines 189–203) renders the branded GSD header. It builds a `headerContent` string from logo lines (green via `theme.fg("success", ...)`) and a tagline ("Get Shit Done vX.Y.Z" via `theme.bold` + `theme.fg("dim", ...)`), then passes it to `ctx.ui.setHeader()` wrapped in a `Text` component from `@gsd/pi-tui`.

Adding the welcome message is a single-line change: append `\n  ${theme.fg("success", "Hello you beautiful person!")}` to `headerContent` before the `setHeader` call. No new imports, no new dependencies, no test changes.

## Recommendation

Add one line after `titleLine` is constructed, appending the welcome message to `headerContent`. Use `theme.fg("success", ...)` to match the logo's green color, keeping it visually consistent. Two leading spaces for alignment with the tagline.

## Don't Hand-Roll

No external solutions needed — this is a one-line string append.

## Existing Code and Patterns

- `src/resources/extensions/gsd/index.ts:189-203` — `session_start` hook renders the header; new line goes inside this try block
- `src/resources/extensions/gsd/index.ts:195-196` — existing theme styling patterns: `theme.fg("success", text)` for green, `theme.bold(text)` for bold, `theme.fg("dim", text)` for dimmed
- `src/resources/extensions/gsd/index.ts:82-88` — `GSD_LOGO_LINES` array, not modified

## Constraints

- Must be inside the existing try/catch so RPC mode (no TUI) gracefully skips it
- `Text` component constructor: `new Text(content, paddingTop, paddingBottom)` — padding values already set, no change needed

## Common Pitfalls

- **Breaking RPC mode** — the welcome message must stay inside the existing try/catch block; any code outside it could crash headless sessions

## Open Risks

None. Single-line change in a well-understood hook with no test coverage to update.

## Requirement Coverage

| Requirement | Role | How This Slice Delivers |
|-------------|------|------------------------|
| R001 | primary owner | Adds "Hello you beautiful person!" to the session_start header output |

## Skills Discovered

No relevant professional skills needed — this is a single-line change to an existing TypeScript file using project-internal APIs only.

## Sources

- Direct code inspection of `src/resources/extensions/gsd/index.ts`
