# M002: Unique Milestone ID Format Swap — Context

**Gathered:** 2026-03-14
**Status:** Ready for planning

## Project Description

Swap the unreleased unique milestone ID format from `M-{rand6}-{seq}` (e.g. `M-abc123-001`) to `M{seq}-{rand6}` (e.g. `M001-abc123`). The old format was never released, so no migration or backwards compatibility is needed — clean replacement everywhere.

## Why This Milestone

The user saw the `M-abc123-001` format before release and identified a better alternative: `M001-abc123`. The new format is superior because the sequential number leads (natural directory sort, pattern consistency with plain `M001`), and the random suffix follows. Since nothing has been released, this is a clean swap with no migration cost.

## User-Visible Outcome

### When this milestone is complete, the user can:

- Enable `unique_milestone_ids: true` and get `M001-abc123` format IDs (not `M-abc123-001`)
- See zero traces of the old `M-{rand6}-{seq}` format anywhere in the codebase

### Entry point / environment

- Entry point: `/gsd` CLI command, `unique_milestone_ids` preference
- Environment: local dev
- Live dependencies involved: none

## Completion Class

- Contract complete means: all tests pass with new format, grep confirms zero old-format traces
- Integration complete means: `deriveState`, `indexWorkspace`, branch operations all work with `M001-abc123` directories
- Operational complete means: none

## Final Integrated Acceptance

To call this milestone complete, we must prove:

- `nextMilestoneId([], true)` returns `M001-{rand6}` format
- All three test suites pass with new-format test data
- `grep -rn 'M(?:-' src/resources/extensions/gsd/` returns zero hits (no old regex pattern traces)
- `grep -rn 'M-[a-z0-9]\{6\}-' src/resources/extensions/gsd/` returns zero hits (no old format examples)

## Risks and Unknowns

- Low risk overall — this is a mechanical format swap on unreleased code
- Regex complexity: the new pattern `M\d+(?:-[a-z0-9]{6})?` is actually simpler than the old one

## Existing Codebase / Prior Art

- `src/resources/extensions/gsd/guided-flow.ts` — ID primitives: `MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId`, `generateMilestonePrefix`, `nextMilestoneId`
- `src/resources/extensions/gsd/state.ts` — `findMilestoneIds` regex, title stripping
- `src/resources/extensions/gsd/workspace-index.ts` — `findMilestoneIds` regex, title stripping
- `src/resources/extensions/gsd/files.ts` — `findMilestoneIds` regex, title stripping
- `src/resources/extensions/gsd/dispatch-guard.ts` — directory scanning regex
- `src/resources/extensions/gsd/worktree.ts` — `SLICE_BRANCH_RE`
- `src/resources/extensions/gsd/worktree-command.ts` — milestone directory filter regex
- `src/resources/extensions/gsd/index.ts` — `MILESTONE_CONTEXT_RE`, prompt dispatch regexes
- `src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — 63 assertions, custom runner
- `src/resources/extensions/gsd/tests/regex-hardening.test.ts` — 83 assertions, vitest wrapper
- `src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — 62 assertions, custom runner
- `src/resources/extensions/gsd/docs/preferences-reference.md` — field documentation
- `src/resources/extensions/gsd/prompts/system.md` — naming convention section
- `src/resources/extensions/gsd/templates/preferences.md` — template (no format-specific content)

> See `.gsd/DECISIONS.md` for all architectural and pattern decisions.

## Scope

### In Scope

- Change `nextMilestoneId()` generation from `M-{prefix}-{seq}` to `M{seq}-{prefix}`
- Update `MILESTONE_ID_RE` from `M(?:-[a-z0-9]{6}-)?\d{3}` to `M\d{3}(?:-[a-z0-9]{6})?`
- Update `extractMilestoneSeq` and `parseMilestoneId` for new format
- Update all 16 production regex sites across 8 files
- Update all ~112 test references across 3 test files
- Update 2 documentation references
- Remove all traces of the old `M-{rand6}-{seq}` format

### Out of Scope / Non-Goals

- Migration tooling for old format (never released)
- Backwards compatibility with old format
- New features beyond the format change

## Technical Constraints

- Must use the same `crypto.randomInt()` approach for random generation (D002)
- Must preserve the `milestoneIdSort` comparator behavior (sort by sequential number)
- Must preserve the `unique_milestone_ids` preference toggle behavior

## Integration Points

- None — purely internal code change

## Open Questions

- None — scope is clear and mechanical
