# M003-alcfdr: Welcome Message

**Vision:** Display "Hello you beautiful person!" as a welcome message every time GSD starts a session.

## Success Criteria

- GSD displays "Hello you beautiful person!" on every session start
- Message is visible in the TUI header area alongside the existing logo and version

## Key Risks / Unknowns

- None — straightforward single-file change in a well-understood hook

## Proof Strategy

- No major risks to retire

## Verification Classes

- Contract verification: visual inspection of session_start output
- Integration verification: none
- Operational verification: none
- UAT / human verification: start GSD and confirm the message appears

## Slices

- [x] **S01: Welcome message in session header** `risk:low` `depends:[]`
  > After this: GSD greets the user with "Hello you beautiful person!" on every session start.

## Boundary Map

### S01
Produces:
  src/resources/extensions/gsd/index.ts → updated session_start hook with welcome message line

Consumes: nothing (leaf node)

## Milestone Definition of Done

This milestone is complete only when all are true:

- S01 deliverable is complete
- "Hello you beautiful person!" is visible in the GSD TUI header on session start
- Success criteria are re-checked against live behavior

## Requirement Coverage

- Covers: R001
