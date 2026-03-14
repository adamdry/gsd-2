---
estimated_steps: 2
estimated_files: 1
---

# T01: Add welcome message to session_start header

**Slice:** S01 — Welcome message in session header
**Milestone:** M003-alcfdr

## Description

Add a single line to the `session_start` hook that appends "Hello you beautiful person!" to the header content, styled green with `theme.fg("success", ...)` to match the logo. This delivers R001.

## Steps

1. In `src/resources/extensions/gsd/index.ts`, locate the `session_start` hook where `headerContent` is built (~line 195-198). After the `titleLine` is appended, add: `headerContent += \`\n  ${theme.fg("success", "Hello you beautiful person!")}\``
2. Verify: run `grep -n "Hello you beautiful person" src/resources/extensions/gsd/index.ts` and confirm the project compiles with `npx tsc --noEmit` (or the project's build command).

## Must-Haves

- [ ] `headerContent` includes "Hello you beautiful person!" styled with `theme.fg("success", ...)`
- [ ] New line is inside the existing try/catch block (RPC safety)
- [ ] Project compiles without errors

## Verification

- `grep -n "Hello you beautiful person" src/resources/extensions/gsd/index.ts` exits 0
- TypeScript compilation succeeds

## Inputs

- `src/resources/extensions/gsd/index.ts` — existing `session_start` hook with header rendering (lines 189-203)
- S01-RESEARCH.md — confirmed the exact insertion point and styling pattern

## Expected Output

- `src/resources/extensions/gsd/index.ts` — modified with one new line appending the welcome message to `headerContent`

## Observability Impact

- **Signal added:** A new visible text line ("Hello you beautiful person!") in the TUI header, rendered green via `theme.fg("success", ...)`.
- **Inspection:** A future agent can verify this task by grepping for the string in source or observing the TUI header at session start.
- **Failure state:** No new failure modes — the welcome message is inside the existing try/catch that already handles RPC mode gracefully. If the header rendering throws, the entire header (logo + title + welcome) is skipped silently.
