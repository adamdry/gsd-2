# S01: Green ASCII Art — Research

**Date:** 2026-03-14

## Summary

Three render sites color the GSD ASCII logo. Each uses a different coloring mechanism — raw ANSI codes, picocolors, and the theme API. All three currently apply cyan/accent and need to switch to green. The changes are mechanical and isolated; no shared state, no test implications beyond a build check.

The only subtlety is the TUI header in `index.ts`, which uses `theme.fg("accent")`. Per D001, we must not use the `accent` semantic (it's shared across all accent-colored UI). The theme's `"success"` color maps to green (`#b5bd68`) and is the right fit — it stays within the theme system so custom themes render consistently, while avoiding the accent semantic entirely.

## Recommendation

Straight color swap in three files. Use the idiomatic coloring mechanism each site already uses — just change the color name/code from cyan to green. Update the `renderLogo()` docstring in `logo.ts` to reflect green instead of cyan in its example.

## Don't Hand-Roll

No external libraries needed. All three coloring mechanisms (raw ANSI, picocolors, theme API) are already in use.

## Existing Code and Patterns

- `src/logo.ts` — shared logo lines + `renderLogo(color)` helper. The `color` param is a function `(s: string) => string`. Docstring example references cyan — update it to green.
- `src/loader.ts:23-34` — first-launch banner. Defines `cyan = '\x1b[36m'` and `colorCyan` function, passes to `renderLogo()`. Note: the file also defines `green = '\x1b[32m'` already (used for "Welcome." text on line 37). Reuse that constant.
- `src/onboarding.ts:181` — onboarding intro. Calls `renderLogo(pc.cyan)`. Swap to `pc.green`.
- `src/resources/extensions/gsd/index.ts:82-89,195` — TUI header. Has its own copy of logo lines (not imported from `logo.ts`). Colors with `theme.fg("accent", line)`. Swap to `theme.fg("success", line)`.

## Constraints

- `theme.fg()` only accepts registered semantic color names — throws on unknowns. Valid green option: `"success"` (maps to `"green"` → `#b5bd68` in dark theme).
- `index.ts` has a duplicated copy of logo lines, not imported from `src/logo.ts`. This is expected — it's a bundled extension under `src/resources/extensions/` with a different module context.
- D001 prohibits using `theme.fg("accent")` for the logo — accent is a shared semantic that would change other UI if modified.

## Common Pitfalls

- **Using `theme.fg("accent")` or changing the accent color globally** — D001 explicitly prohibits this. Only the logo color changes; accent remains cyan-ish for all other UI.
- **Forgetting the docstring in `logo.ts`** — line 22 references `\x1b[36m` (cyan) and `picocolors.cyan` as examples. Should reflect green.

## Open Risks

None. All three changes are single-line edits with no behavioral dependencies.

## Skills Discovered

| Technology | Skill | Status |
|------------|-------|--------|
| ANSI colors / picocolors | N/A | none needed — trivial color constants |

## Sources

- `pkg/dist/modes/interactive/theme/dark.json` — theme color mappings (source: local file)
- `pkg/dist/modes/interactive/theme/theme.js:266-271` — `theme.fg()` implementation, throws on unknown color names (source: local file)
