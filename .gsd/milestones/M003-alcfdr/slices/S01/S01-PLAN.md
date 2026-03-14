# S01: Welcome message in session header

**Goal:** GSD displays "Hello you beautiful person!" in the TUI header on every session start.
**Demo:** Launch GSD and see the welcome message below the tagline in green.

## Must-Haves

- "Hello you beautiful person!" appears in the session header on every start
- Message uses `theme.fg("success", ...)` for green color, consistent with logo styling
- Code stays inside the existing try/catch so RPC mode is unaffected

## Verification

- `grep -n "Hello you beautiful person" src/resources/extensions/gsd/index.ts` exits 0
- Visual confirmation: start GSD and observe the message in the header
- Diagnostic check: if the message is absent at runtime, confirm the try/catch around header rendering isn't throwing by temporarily adding a `console.log` or checking that the logo still renders (same code path)

## Tasks

- [x] **T01: Add welcome message to session_start header** `est:10m`
  - Why: Delivers R001 — the welcome greeting on session start
  - Files: `src/resources/extensions/gsd/index.ts`
  - Do: After `titleLine` is appended to `headerContent` (~line 197), append `\n  ${theme.fg("success", "Hello you beautiful person!")}` to `headerContent`. Must remain inside the existing try/catch block.
  - Verify: `grep -n "Hello you beautiful person" src/resources/extensions/gsd/index.ts` exits 0; build succeeds (`npx tsc --noEmit` or equivalent)
  - Done when: The welcome message line exists in `session_start`, grep confirms it, and the project compiles without errors

## Observability / Diagnostics

- **Runtime signal:** The welcome message is rendered via `theme.fg("success", ...)` inside the existing `session_start` try/catch. If it fails, the catch block silently skips header rendering — same as the existing logo behavior. No new failure modes introduced.
- **Inspection surface:** `grep -rn "Hello you beautiful person" src/` confirms presence in source. At runtime, the message is visible in the TUI header on every session start.
- **Failure visibility:** If the message doesn't appear, the try/catch swallows the error silently (by design — RPC mode). An agent can verify by grepping the source or launching GSD and checking the header visually.
- **Redaction:** No secrets or credentials involved.

## Files Likely Touched

- `src/resources/extensions/gsd/index.ts`
