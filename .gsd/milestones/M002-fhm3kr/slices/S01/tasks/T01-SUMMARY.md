---
id: T01
parent: S01
milestone: M002-fhm3kr
provides:
  - New milestone ID format M{seq}-{rand6} in all production code, tests, and docs
key_files:
  - src/resources/extensions/gsd/guided-flow.ts
  - src/resources/extensions/gsd/state.ts
  - src/resources/extensions/gsd/workspace-index.ts
  - src/resources/extensions/gsd/files.ts
  - src/resources/extensions/gsd/dispatch-guard.ts
  - src/resources/extensions/gsd/worktree.ts
  - src/resources/extensions/gsd/worktree-command.ts
  - src/resources/extensions/gsd/index.ts
  - src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts
  - src/resources/extensions/gsd/tests/regex-hardening.test.ts
  - src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts
  - src/resources/extensions/gsd/docs/preferences-reference.md
  - src/resources/extensions/gsd/prompts/system.md
key_decisions:
  - Old-format rejection test string M-abc123-001 retained in unique-milestone-ids.test.ts as intentional negative test case
patterns_established:
  - New ID format M{seq}-{rand6} e.g. M001-abc123 (seq in group 1, prefix in group 2)
  - MILESTONE_ID_RE /^M\d{3}(?:-[a-z0-9]{6})?$/ is the single source of truth for format validation
observability_surfaces:
  - none — format swap only, existing failure paths preserved
duration: ~45min (across two sessions with crash recovery)
verification_result: passed
completed_at: 2026-03-14
blocker_discovered: false
---

# T01: Replace milestone ID format across all production, test, and doc files

**Swapped milestone ID format from `M-{rand6}-{seq}` to `M{seq}-{rand6}` across 8 production files, 3 test files, and 2 doc files with zero old-format traces remaining.**

## What Happened

Updated all 5 core primitives in `guided-flow.ts`: `MILESTONE_ID_RE` now validates `M\d{3}(?:-[a-z0-9]{6})?`, `extractMilestoneSeq` captures digits from group 1 (`M(\d{3})`), `parseMilestoneId` returns `{ prefix: m[2], num: parseInt(m[1]) }` (swapped capture groups), `findMilestoneIds` scans directories with new regex, and `nextMilestoneId` generates `` `M${seq}-${prefix}` `` format.

Updated regex patterns in 7 remaining production files (`state.ts`, `workspace-index.ts`, `files.ts`, `dispatch-guard.ts`, `worktree.ts`, `worktree-command.ts`, `index.ts`) — each pattern swapped from `M-[a-z0-9]{6}-\d{3}` to `M\d{3}(?:-[a-z0-9]{6})?` or equivalent.

Migrated all test data across 3 test files: `M-abc123-001` → `M001-abc123` style across ~114 references. Updated regex match/reject cases, parse expectations (group swap), sort arrays, and assertion patterns.

Updated 2 doc files with new format descriptions and examples.

## Verification

- `npx vitest run unique-milestone-ids.test.ts` — **63 assertions passed** ✅
- `npx vitest run regex-hardening.test.ts` — **82 assertions passed** ✅
- `npx vitest run integration-mixed-milestones.test.ts` — **62 assertions passed** ✅
- Total: **207 assertions passed**, 0 failed
- `grep -rn 'M(?:-' src/resources/extensions/gsd/` — **zero hits** ✅
- `grep -rn 'M-[a-z0-9]\{6\}-\|M-abc123-\|M-eh88as-\|M-w2lhcz-'` — **1 hit**: intentional rejection test in unique-milestone-ids.test.ts line 75 (`!MILESTONE_ID_RE.test('M-abc123-001')` verifies old format is rejected) ✅
- `parseMilestoneId('invalid')` returns `undefined` — failure path preserved ✅
- `MILESTONE_ID_RE.test('M001-abc123')` → true, validates new format ✅
- `extractMilestoneSeq('M001-abc123')` → `'001'` ✅
- `parseMilestoneId('M001-abc123')` → `{ prefix: 'abc123', num: 1 }` ✅

## Diagnostics

No new observability surfaces — this is a pure format swap. Existing failure paths are preserved: `parseMilestoneId` returns `undefined` on invalid input, `extractMilestoneSeq` returns `undefined` on non-match. Grep for `M\d+(?:-[a-z0-9]{6})?` confirms new format is in place.

## Deviations

None.

## Known Issues

None.

## Files Created/Modified

- `src/resources/extensions/gsd/guided-flow.ts` — Core primitives: MILESTONE_ID_RE, extractMilestoneSeq, parseMilestoneId, findMilestoneIds, nextMilestoneId
- `src/resources/extensions/gsd/state.ts` — Regex patterns for milestone detection
- `src/resources/extensions/gsd/workspace-index.ts` — Regex patterns for workspace indexing
- `src/resources/extensions/gsd/files.ts` — Regex patterns for file operations
- `src/resources/extensions/gsd/dispatch-guard.ts` — Regex pattern for dispatch guard
- `src/resources/extensions/gsd/worktree.ts` — SLICE_BRANCH_RE with updated group numbering
- `src/resources/extensions/gsd/worktree-command.ts` — Branch name regex pattern
- `src/resources/extensions/gsd/index.ts` — Regex patterns for milestone context and detection
- `src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — All test data migrated to new format
- `src/resources/extensions/gsd/tests/regex-hardening.test.ts` — All pattern strings and test data migrated
- `src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — All fixture data migrated
- `src/resources/extensions/gsd/docs/preferences-reference.md` — Updated format example
- `src/resources/extensions/gsd/prompts/system.md` — Updated format description
