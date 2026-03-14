# S01: Welcome message in session header — UAT

**Milestone:** M003-alcfdr
**Written:** 2026-03-14

## UAT Type

- UAT mode: mixed
- Why this mode is sufficient: Source verification confirms the code exists and compiles; runtime verification confirms it renders visually as intended.

## Preconditions

- GSD is built from the branch containing S01 changes
- Terminal supports ANSI color output
- No RPC-only mode active (RPC mode skips header rendering by design)

## Smoke Test

Run `grep -n "Hello you beautiful person" src/resources/extensions/gsd/index.ts` — should exit 0 and show the string at line 198 inside the `headerContent` template literal.

## Test Cases

### 1. Welcome message appears in TUI header on session start

1. Open a terminal
2. Launch GSD (e.g. `pi` or the project's start command)
3. Observe the TUI header area
4. **Expected:** "Hello you beautiful person!" appears below the tagline, rendered in green text

### 2. Welcome message appears on every session start (not just first launch)

1. Launch GSD and confirm the welcome message is visible
2. Exit GSD
3. Launch GSD again
4. **Expected:** "Hello you beautiful person!" appears again in the header — not conditional on first-run state

### 3. Source contains the welcome message string

1. Run: `grep -rn "Hello you beautiful person" src/resources/extensions/gsd/index.ts`
2. **Expected:** Output shows the string at line 198, inside the `headerContent` template literal, wrapped in `theme.fg("success", ...)`

### 4. Project compiles without errors

1. Run: `npx tsc --noEmit`
2. **Expected:** Clean exit, no type errors

## Edge Cases

### RPC mode does not crash

1. If GSD supports RPC mode, start a session in RPC mode
2. **Expected:** No crash or error. The welcome message may not be visible (header rendering is skipped in RPC mode), but the process should function normally.

## Failure Signals

- Welcome message absent from TUI header despite the string being present in source → the try/catch around header rendering is likely throwing. Check that the logo also renders (same code path).
- `grep` finds no match → the change was lost or reverted. Check the branch.
- `npx tsc --noEmit` fails → type error introduced. Check the template literal syntax.

## Requirements Proved By This UAT

- R001 — Test cases 1 and 2 prove the welcome message displays on every session start. Test case 3 proves the source contains the implementation. Test case 4 proves it compiles.

## Not Proven By This UAT

- Exact color rendering (green) depends on terminal capabilities and theme configuration — not verifiable in automated tests.

## Notes for Tester

- The message is rendered via `theme.fg("success", ...)` which produces green in the default theme. If using a custom theme with a different success color, the text will still appear but may not be green.
- The try/catch around header rendering means failures are silent — if the message doesn't appear, check that the logo renders too (same block).
