---
estimated_steps: 5
estimated_files: 4
---

# T01: Swap logo color from cyan to green in all render sites

**Slice:** S01 — Green ASCII Art
**Milestone:** M001

## Description

Change the GSD ASCII art logo color from cyan/light blue to green in all three render contexts: first-launch loader (raw ANSI), onboarding wizard (picocolors), and TUI header (theme API). Also update the `renderLogo()` docstring in `logo.ts` to reflect green.

## Steps

1. In `src/loader.ts`: find the `renderLogo(colorCyan)` call (or equivalent cyan function). Replace with the existing green color function — the file already defines `green = '\x1b[32m'`. Rename/reuse accordingly. Remove the cyan constant if it becomes unused.
2. In `src/onboarding.ts`: find `renderLogo(pc.cyan)` and change to `renderLogo(pc.green)`.
3. In `src/resources/extensions/gsd/index.ts`: find `theme.fg("accent", line)` in the logo rendering block and change to `theme.fg("success", line)`.
4. In `src/logo.ts`: update the docstring/comment on `renderLogo()` that references `\x1b[36m` or `picocolors.cyan` — change examples to reference green equivalents.
5. Run `npm run build` and verify no compilation errors.

## Must-Haves

- [ ] `src/loader.ts` renders logo in green (ANSI `\x1b[32m`)
- [ ] `src/onboarding.ts` renders logo with `pc.green`
- [ ] `src/resources/extensions/gsd/index.ts` renders logo with `theme.fg("success")`
- [ ] `src/logo.ts` docstring reflects green, not cyan
- [ ] No cyan color codes remain in logo-rendering code paths
- [ ] Build passes

## Verification

- `grep -rn 'colorCyan\|pc\.cyan\|"accent"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts` returns no matches in logo lines
- `npm run build` exits 0

## Observability Impact

- **No new runtime signals.** This task swaps static color constants — no new logs, error paths, or state transitions are introduced.
- **Future inspection:** A future agent verifies this task by grepping for `green`/`\x1b[32m`/`pc.green`/`"success"` in the four target files and confirming no cyan remnants in logo-rendering lines. Build passing confirms type-level correctness.
- **No failure state changes.** The logo render paths have no error handling that changes; color functions are infallible.

## Inputs

- S01-RESEARCH.md — line numbers and exact patterns for each file
- D001 — do not use `theme.fg("accent")` for the logo

## Expected Output

- `src/loader.ts` — logo colored with green ANSI code
- `src/onboarding.ts` — logo colored with `pc.green`
- `src/resources/extensions/gsd/index.ts` — logo colored with `theme.fg("success")`
- `src/logo.ts` — docstring updated to reference green
