# M001: Green ASCII Art — Context

**Gathered:** 2026-03-14
**Status:** Ready for planning

## Project Description

Change the GSD ASCII art logo color from cyan/light blue to green across all render sites.

## Why This Milestone

User wants the logo green. Currently it renders cyan in three places.

## User-Visible Outcome

### When this milestone is complete, the user can:

- See the GSD ASCII art in green on first launch (loader)
- See the GSD ASCII art in green during onboarding
- See the GSD ASCII art in green in the TUI header

### Entry point / environment

- Entry point: CLI launch (`gsd` command)
- Environment: local dev terminal
- Live dependencies involved: none

## Completion Class

- Contract complete means: all three render sites use green instead of cyan
- Integration complete means: n/a — pure cosmetic change
- Operational complete means: n/a

## Final Integrated Acceptance

To call this milestone complete, we must prove:

- The logo renders green in `loader.ts` (ANSI `\x1b[32m`)
- The logo renders green in `onboarding.ts` (`pc.green`)
- The logo renders green in `index.ts` (direct green, not theme accent)

## Risks and Unknowns

- None — straightforward color swap

## Existing Codebase / Prior Art

- `src/logo.ts` — raw logo lines and `renderLogo()` helper
- `src/loader.ts` — first-launch banner, uses raw ANSI cyan `\x1b[36m`
- `src/onboarding.ts` — onboarding intro, uses `pc.cyan`
- `src/resources/extensions/gsd/index.ts` — TUI header, uses `theme.fg("accent", line)` which maps to the theme's accent color (currently cyan-ish)

> See `.gsd/DECISIONS.md` for all architectural and pattern decisions.

## Relevant Requirements

- R001 — GSD ASCII art renders in green (this milestone delivers it entirely)

## Scope

### In Scope

- Change logo color from cyan to green in loader.ts, onboarding.ts, and index.ts

### Out of Scope / Non-Goals

- Changing the ASCII art design itself
- Changing the theme accent color globally
- Changing any other UI colors

## Technical Constraints

- The TUI header in index.ts currently uses `theme.fg("accent")` — this is a shared theme semantic. Changing it to green must not use the accent semantic (which would change other accent-colored UI). Instead, use a direct green color or `theme.fg("success")` if that maps to green, or raw ANSI.

## Integration Points

- None

## Open Questions

- None
