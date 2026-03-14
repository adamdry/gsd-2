# S01: Green ASCII Art — UAT

**Milestone:** M001
**Written:** 2026-03-14

## UAT Type

- UAT mode: mixed (artifact-driven grep + human-experience visual check)
- Why this mode is sufficient: The color swap is statically verifiable via grep, but final visual confirmation requires a human seeing green output in a terminal.

## Preconditions

- Repository is built (`npm run build` passes)
- A terminal with ANSI color support (most modern terminals)
- For visual tests: access to run `npx tsx src/loader.ts` or launch GSD in a fresh directory

## Smoke Test

Run `grep -n 'colorGreen\|pc\.green\|"success"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts` and confirm all three files have green/success at their logo-rendering lines.

## Test Cases

### 1. No cyan remnants in logo rendering code

1. Run: `grep -n 'colorCyan\|"accent"' src/loader.ts src/onboarding.ts src/resources/extensions/gsd/index.ts src/logo.ts`
2. **Expected:** Zero matches. (Note: `pc.cyan` may appear in `onboarding.ts` for non-logo uses — OAuth URL display, type definition, fallback identity — those are acceptable.)

### 2. Green codes present in loader

1. Run: `grep -n 'colorGreen\|\\x1b\[32m' src/loader.ts`
2. **Expected:** Matches at line 23 (`green` ANSI constant) and line 26 (`colorGreen` function). The logo renders using `renderLogo(colorGreen)`.

### 3. Green codes present in onboarding

1. Run: `grep -n 'renderLogo(pc\.green)' src/onboarding.ts`
2. **Expected:** Match at line 181. The onboarding wizard logo uses `pc.green`.

### 4. Success theme in TUI header logo

1. Run: `grep -n '"success"' src/resources/extensions/gsd/index.ts`
2. **Expected:** Match at line 195 in the logo rendering block: `theme.fg("success", line)`.

### 5. Docstring updated in logo.ts

1. Run: `grep -n 'green\|\\x1b\[32m' src/logo.ts`
2. **Expected:** Match at line 22 referencing `\x1b[32m` and `picocolors.green` in the `@param color` docstring.

### 6. Build passes

1. Run: `npm run build`
2. **Expected:** Exit code 0, no type errors.

### 7. Visual: Loader logo is green

1. Delete or rename `~/.gsd/` to simulate first launch.
2. Run GSD in a project directory.
3. **Expected:** The ASCII art "GSD" logo in the loader banner renders in green text, not cyan/light blue.
4. Restore `~/.gsd/` afterward.

### 8. Visual: TUI header logo is green

1. Launch GSD in an existing project.
2. Type `/gsd` to open the GSD wizard/dashboard.
3. **Expected:** The ASCII art logo in the TUI header renders in green.

## Edge Cases

### Non-logo cyan usage is preserved

1. Run: `grep -n 'pc\.cyan' src/onboarding.ts`
2. **Expected:** Matches at lines 28 (type def), 116 (fallback identity), and 341 (OAuth URL display). These are non-logo uses and must remain cyan.

### Theme accent unchanged globally

1. Run: `grep -rn 'theme\.fg("accent"' src/resources/extensions/gsd/`
2. **Expected:** Zero matches specifically for the logo block. Other accent uses elsewhere in the extension are fine and expected — only the logo line changed.

## Failure Signals

- Cyan/light-blue logo text visible in loader, onboarding, or TUI header
- `grep` finds `colorCyan`, `pc.cyan` in logo-rendering lines, or `"accent"` in the logo block of `index.ts`
- `npm run build` fails with type errors related to color functions

## Requirements Proved By This UAT

- R001 — GSD ASCII art renders in green across all three contexts (loader, onboarding, TUI header)

## Not Proven By This UAT

- Visual tests 7 and 8 require human execution in a real terminal — grep verification alone cannot confirm perceived color.
- Onboarding visual test is omitted because triggering onboarding requires a clean `~/.gsd/` state and interactive prompts — impractical for automated testing.

## Notes for Tester

- The color change is purely cosmetic. No behavioral changes, no new error paths.
- If your terminal doesn't support ANSI colors, the logo will appear uncolored regardless — test in a standard terminal emulator.
