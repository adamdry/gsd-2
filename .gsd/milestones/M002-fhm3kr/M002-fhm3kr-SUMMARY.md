---
id: M002-fhm3kr
provides:
  - Unique milestone IDs in `M{seq}-{rand6}` format (e.g. `M001-abc123`)
  - Updated regex pattern `M\d+(?:-[a-z0-9]{6})?` across all production code
key_decisions:
  - "D007: Format changed from `M-{rand6}-{seq}` to `M{seq}-{rand6}` — sequential number leads for natural sort"
  - "D008: Regex pattern simplified to `M\\d+(?:-[a-z0-9]{6})?`"
patterns_established:
  - Vitest detection guard uses nullish check before `in` operator
observability_surfaces:
  - Test suites: 62 + 82 + 62 = 206 passing assertions across 3 test files
  - Grep verification: zero old-format traces in production code
requirement_outcomes: []
duration: single session
verification_result: passed
completed_at: 2026-03-14T08:05:00.000Z
---

# M002-fhm3kr: Unique Milestone ID Format Swap

**Swapped unreleased unique milestone ID format from `M-{rand6}-{seq}` to `M{seq}-{rand6}` — clean replacement with zero old-format traces remaining.**

## What Happened

Single-slice mechanical format swap across the entire GSD extension codebase. The old `M-abc123-001` format (implemented in M001 but never released) was replaced with `M001-abc123` everywhere: ID generation primitives in `guided-flow.ts`, regex patterns across 8 production files, test data in 3 test files, and format descriptions in 2 doc files.

Key changes:
- `nextMilestoneId()` now generates `M{seq}-{prefix}` (was `M-{prefix}-{seq}`)
- `MILESTONE_ID_RE` is now `/^M\d{3}(?:-[a-z0-9]{6})?$/` (was `/^M(?:-[a-z0-9]{6}-)?\d{3}$/`)
- `extractMilestoneSeq` and `parseMilestoneId` updated to parse digits immediately after `M`
- All production regex sites use `M\d+(?:-[a-z0-9]{6})?` (was `M(?:-[a-z0-9]{6}-)?\d+`)

During milestone completion, a vitest detection bug was discovered and fixed in the two custom-runner test files — the `in` operator was applied to a potentially `undefined` value from optional chaining.

## Cross-Slice Verification

Single slice — no cross-slice integration needed. All success criteria verified:

| Criterion | Evidence |
|-----------|----------|
| `nextMilestoneId([], true)` generates `M001-{rand6}` format | `guided-flow.ts` line: `` return `M${seq}-${generateMilestonePrefix()}`; `` |
| All regex/parser sites use `M\d+(?:-[a-z0-9]{6})?` pattern | `MILESTONE_ID_RE = /^M\d{3}(?:-[a-z0-9]{6})?$/`; grep confirms pattern in all 8 production files |
| Zero traces of old `M(?:-` pattern | `grep -rn 'M(?:-' src/resources/extensions/gsd/` → exit 1 (no matches) |
| Zero traces of old `M-abc123-` style examples | `grep -rn 'M-[a-z0-9]\{6\}-' src/resources/extensions/gsd/` → exit 1 (no matches) |
| All three test suites pass | unique-milestone-ids: 62 passed; regex-hardening: 82 passed; integration-mixed-milestones: 62 passed |
| No regression in existing tests | GSD vitest suite: 3 passed (the 3 milestone-related files); broader failures are pre-existing unrelated import issues |
| Docs updated | `preferences-reference.md` and `system.md` both show `M{seq}-{rand6}` format |

## Requirement Changes

No requirements tracked — operating in legacy compatibility mode (no `REQUIREMENTS.md`).

## Forward Intelligence

### What the next milestone should know
- The unique milestone ID feature is now fully implemented and tested but still gated behind `unique_milestone_ids: true` preference — it defaults to `false`
- The vitest detection pattern in custom-runner test files was buggy; the fix applied here should be used as the template for any future custom-runner tests

### What's fragile
- Custom-runner test files (those using `main()` instead of vitest's `test()`) don't get picked up by `npx vitest run` — they must be run via `npx tsx` separately. This is a known project-wide pattern, not specific to M002.

### Authoritative diagnostics
- `npx tsx src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — 62 assertions covering all ID primitives
- `npx vitest run src/resources/extensions/gsd/tests/regex-hardening.test.ts` — 82 assertions covering all regex sites
- `npx tsx src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — 62 assertions covering state derivation, workspace indexing, dispatch guard, and branch ops

### What assumptions changed
- No assumptions changed — the scope was clear and mechanical as predicted

## Files Created/Modified

- `src/resources/extensions/gsd/guided-flow.ts` — updated `MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId`, `nextMilestoneId` for new format
- `src/resources/extensions/gsd/state.ts` — updated `findMilestoneIds` regex and title stripping
- `src/resources/extensions/gsd/workspace-index.ts` — updated `findMilestoneIds` regex and title stripping
- `src/resources/extensions/gsd/files.ts` — updated `findMilestoneIds` regex and title stripping
- `src/resources/extensions/gsd/dispatch-guard.ts` — updated directory scanning regex
- `src/resources/extensions/gsd/worktree.ts` — updated `SLICE_BRANCH_RE`
- `src/resources/extensions/gsd/worktree-command.ts` — updated milestone directory filter regex
- `src/resources/extensions/gsd/index.ts` — updated `MILESTONE_CONTEXT_RE` and prompt dispatch regexes
- `src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — new-format test data + vitest detection fix
- `src/resources/extensions/gsd/tests/regex-hardening.test.ts` — new-format test data
- `src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — new-format test data + vitest detection fix
- `src/resources/extensions/gsd/docs/preferences-reference.md` — updated format description
- `src/resources/extensions/gsd/prompts/system.md` — updated naming convention example
