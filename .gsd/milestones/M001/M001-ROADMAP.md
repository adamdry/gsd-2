# M001: Green ASCII Art

**Vision:** Change the GSD ASCII art logo color from cyan/light blue to green across all render contexts.

## Success Criteria

- GSD ASCII art renders in green on first launch
- GSD ASCII art renders in green during onboarding
- GSD ASCII art renders in green in TUI header

## Key Risks / Unknowns

None — straightforward color swap in three files.

## Proof Strategy

N/A — no significant risks.

## Verification Classes

- Contract verification: grep for cyan/`\x1b[36m`/`pc.cyan` confirms no remnants; green codes present
- Integration verification: none
- Operational verification: none
- UAT / human verification: visual confirmation that logo is green

## Milestone Definition of Done

This milestone is complete only when all are true:

- All three render sites use green instead of cyan
- No cyan color codes remain in logo rendering paths
- Build passes

## Requirement Coverage

- Covers: R001
- Partially covers: none
- Leaves for later: none
- Orphan risks: none

## Slices

- [x] **S01: Green ASCII art** `risk:low` `depends:[]`
  > After this: GSD logo renders in green instead of light blue in loader, onboarding, and TUI header.

## Boundary Map

### S01

Produces:
- Green logo rendering in `src/loader.ts`, `src/onboarding.ts`, `src/resources/extensions/gsd/index.ts`

Consumes:
- nothing (single slice, no dependencies)
