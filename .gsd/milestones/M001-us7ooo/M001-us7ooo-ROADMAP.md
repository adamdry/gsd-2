# M001-us7ooo: Unique Milestone IDs

**Vision:** Enable GSD to generate collision-resistant milestone IDs (`M-{rand6}-{seq}`) as an opt-in preference, while maintaining full backwards compatibility with the legacy sequential format (`M001`). All code paths — directory scanning, state derivation, branch operations, sorting, prompt dispatch, and user-facing documentation — handle both formats transparently.

## Success Criteria

- User can enable `unique_milestone_ids: true` in preferences and all new milestones receive `M-{rand6}-{seq}` format IDs
- Existing `M001`-format milestones continue to work identically — no behavioral regression
- Mixed-format milestone directories (old + new) sort, parse, and display correctly throughout the system
- The `/gsd prefs wizard` includes the unique milestone IDs toggle
- All documentation (preferences reference, template, system prompt) describes the new format
- All regex/parser sites across the codebase accept both ID formats — zero `M\d+`-only patterns remain in production code
- Integration tests prove real filesystem and git operations work with new-format milestone directories

## Key Risks / Unknowns

- Regex sprawl — `M\d+` patterns exist in 12+ locations across 8 files; missing even one causes silent failures with new-format IDs
- Sort correctness — bare `.sort()` on milestone ID arrays produces lexicographic order, which breaks when mixing `M001` and `M-abc123-001` formats
- Dispatch guard — the existing `parseInt(id.slice(1))` approach returns NaN for new-format IDs, silently breaking milestone traversal

## Proof Strategy

- Regex sprawl → retire in S02 by widening all 12 sites and running `grep` to confirm zero old-format-only patterns remain
- Sort correctness → retire in S01 by implementing `milestoneIdSort` comparator and replacing all bare `.sort()` calls in S02
- Dispatch guard → retire in S02 by refactoring from number-parsing to directory-scanning approach, proven by targeted tests in S04

## Verification Classes

- Contract verification: unit tests (S01: `unique-milestone-ids.test.ts`, S02: `regex-hardening.test.ts`), grep for zero remaining `M\d+` patterns
- Integration verification: filesystem + git integration tests in S04 (`integration-mixed-milestones.test.ts`) proving `deriveState()`, `indexWorkspace()`, branch operations with real directories
- Operational verification: none — no service lifecycle involved
- UAT / human verification: run `/gsd prefs wizard` to confirm toggle appears; create a milestone with `unique_milestone_ids: true` to confirm new-format ID is generated

## Milestone Definition of Done

This milestone is complete only when all are true:

- All ID primitives (`MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId`, `milestoneIdSort`, `generateMilestonePrefix`, `nextMilestoneId`) are exported and tested
- `GSDPreferences.unique_milestone_ids` plumbed through interface, validation, merge, and serialization
- All 12 regex/parser sites accept both `M001` and `M-abc123-001` formats — `grep -rn 'M\\d+' src/resources/extensions/gsd/{state,workspace-index,files,worktree,worktree-command,index,dispatch-guard}.ts` returns zero hits
- All 4 bare `.sort()` calls on milestone arrays use `milestoneIdSort`
- `dispatch-guard.ts` uses directory scanning instead of `parseInt(id.slice(1))`
- Wizard toggle, preferences reference, template, and system prompt all document `unique_milestone_ids`
- Integration tests pass with ~60 assertions covering `deriveState`, `indexWorkspace`, `inlinePriorMilestoneSummary`, `getPriorSliceCompletionBlocker`, and branch operations with mixed-format milestone directories
- All existing GSD test suites pass without regression

## Requirement Coverage

> **Note:** No `REQUIREMENTS.md` found — operating in legacy compatibility mode. Requirement coverage cannot be mapped. If requirements are defined elsewhere, they should be consolidated into `.gsd/REQUIREMENTS.md` for future milestones.

- Covers: N/A (no requirements file)
- Partially covers: N/A
- Leaves for later: N/A
- Orphan risks: Cannot assess — no requirements document exists

## Slices

- [x] **S01: ID generation and config plumbing** `risk:high` `depends:[]`
  > After this: unit tests prove `nextMilestoneId()` returns `M-{rand6}-{seq}` when `unique_milestone_ids: true` and `M001`-style when false. Preferences field validates, merges, and serializes correctly. All call sites in `guided-flow.ts` read and pass the preference.

- [x] **S02: Regex hardening and backwards compat** `risk:high` `depends:[S01]`
  > After this: `regex-hardening.test.ts` passes with assertions covering all 12 regex/parser sites and 4 sort fixes. `grep` confirms zero `M\d+`-only patterns remain in production code. `dispatch-guard.ts` uses directory scanning instead of number parsing.

- [x] **S03: UX — wizard toggle and documentation** `risk:low` `depends:[S01]`
  > After this: `/gsd prefs wizard` shows the unique milestone IDs prompt. `preferences-reference.md`, `templates/preferences.md`, and `system.md` all reference `unique_milestone_ids`.

- [x] **S04: Integration tests and end-to-end verification** `risk:medium` `depends:[S01,S02]`
  > After this: ~60 integration test assertions pass, proving `deriveState()`, `indexWorkspace()`, `inlinePriorMilestoneSummary()`, `getPriorSliceCompletionBlocker()`, and branch operations all work with new-format and mixed-format milestone directories through real filesystem and git fixtures.

## Boundary Map

### S01 → S02

Produces:
- `guided-flow.ts` → `MILESTONE_ID_RE` (anchored regex matching both formats)
- `guided-flow.ts` → `extractMilestoneSeq(id)` (returns sequential number from either format, 0 for non-matches)
- `guided-flow.ts` → `parseMilestoneId(id)` (returns `{ prefix?: string, num: number }`)
- `guided-flow.ts` → `milestoneIdSort(a, b)` (comparator for sorting mixed-format arrays)
- `guided-flow.ts` → `generateMilestonePrefix()` (6-char lowercase `[a-z0-9]` via `crypto.randomInt()`)
- `guided-flow.ts` → `nextMilestoneId(milestoneIds, uniqueEnabled?)` (generates new or legacy format)
- `preferences.ts` → `GSDPreferences.unique_milestone_ids` (boolean field, validated, merged, serialized)

Consumes:
- nothing (first slice)

### S01 → S03

Produces:
- `preferences.ts` → `GSDPreferences.unique_milestone_ids` (boolean preference field)
- `guided-flow.ts` → `nextMilestoneId()` (generation function, for documentation reference)

Consumes:
- nothing (first slice)

### S02 → S04

Produces:
- `state.ts` → widened `findMilestoneIds` regex, `milestoneIdSort` import
- `workspace-index.ts` → widened `findMilestoneIds` regex, `milestoneIdSort` import
- `files.ts` → widened regex for `findMilestoneIds` and title stripping
- `dispatch-guard.ts` → refactored to directory-scanning with `extractMilestoneSeq`
- `worktree.ts` → widened `SLICE_BRANCH_RE` to capture new-format milestone IDs
- `worktree-command.ts` → widened branch matching regex
- `index.ts` → widened `hasExistingMilestones` and `MILESTONE_CONTEXT_RE`

Consumes from S01:
- `guided-flow.ts` → `extractMilestoneSeq`, `milestoneIdSort`, `MILESTONE_ID_RE`

### S01 + S02 → S04

Produces:
- All production code paths handling both ID formats (S01 primitives + S02 site widening)

Consumes from S01:
- `guided-flow.ts` → `extractMilestoneSeq`, `milestoneIdSort`
Consumes from S02:
- All widened regex sites in `state.ts`, `workspace-index.ts`, `files.ts`, `dispatch-guard.ts`, `worktree.ts`

### S01 → S03 (docs)

Produces:
- `preferences.ts` → `unique_milestone_ids` field definition

Consumes from S01:
- `GSDPreferences.unique_milestone_ids` (for wizard integration and documentation)
