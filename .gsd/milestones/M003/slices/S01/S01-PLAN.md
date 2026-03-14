# S01: Green ASCII Art

**Goal:** GSD ASCII art logo renders in green instead of cyan/light blue across all three render sites.
**Demo:** `grep` confirms no cyan color codes remain in logo paths; green codes present in all three files. Build passes.

## Must-Haves

- `src/loader.ts` uses green ANSI code instead of cyan for the logo
- `src/onboarding.ts` uses `pc.green` instead of `pc.cyan` for the logo
- `src/resources/extensions/gsd/index.ts` uses `theme.fg("success")` instead of `theme.fg("accent")` for the logo
- `src/logo.ts` docstring reflects green instead of cyan

## Verification

- `grep -n 'cyan\|\\\\x1b\[36m\|pc\.cyan\|"accent"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts src/logo.ts` — returns no matches in logo-coloring lines
- `grep -n 'green\|\\\\x1b\[32m\|pc\.green\|"success"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts src/logo.ts` — returns matches at expected lines
- `npm run build` passes

## Tasks

- [x] **T01: Swap logo color from cyan to green in all render sites** `est:15m`
  - Why: Only task — covers the entire slice (R001)
  - Files: `src/loader.ts`, `src/onboarding.ts`, `src/resources/extensions/gsd/index.ts`, `src/logo.ts`
  - Do: In `loader.ts`, replace `colorCyan` usage in `renderLogo()` call with the existing `green`/`colorGreen` equivalent. In `onboarding.ts`, swap `pc.cyan` → `pc.green` in the `renderLogo()` call. In `index.ts`, swap `theme.fg("accent", line)` → `theme.fg("success", line)` in the logo rendering block. In `logo.ts`, update the docstring example to reference green instead of cyan. Do NOT change the accent color globally (D009).
  - Verify: Grep for cyan remnants (zero matches in logo paths), grep for green presence, `npm run build` passes
  - Done when: All three render sites use green, no cyan remnants in logo code, build clean

## Observability / Diagnostics

- **Visual inspection:** Run GSD in a fresh directory (no `~/.gsd/`) to see the loader logo in green. Run `/gsd` inside a session to see the TUI header logo in green. Trigger onboarding to see the wizard logo in green.
- **Grep verification:** `grep -n 'green\|\\x1b\[32m\|pc\.green\|"success"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts src/logo.ts` confirms green codes present at logo-rendering lines.
- **Negative grep:** `grep -n 'cyan\|\\x1b\[36m\|pc\.cyan\|"accent"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts src/logo.ts` confirms no cyan remnants in logo-coloring lines (other uses of cyan/accent for non-logo purposes are acceptable).
- **Build output:** `npm run build` exits 0 — no type errors from color function changes.
- **No runtime signals change:** This is a static color swap with no new error paths, logs, or state. No runtime diagnostics to add.

## Files Likely Touched

- `src/loader.ts`
- `src/onboarding.ts`
- `src/resources/extensions/gsd/index.ts`
- `src/logo.ts`
