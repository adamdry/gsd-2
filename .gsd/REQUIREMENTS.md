# Requirements

This file is the explicit capability and coverage contract for the project.

## Active

### R001 — No duplicate `findMilestoneIds` definitions
- Class: quality-attribute
- Status: active
- Description: `findMilestoneIds` must be defined exactly once (in `guided-flow.ts`) and imported by all callers. No copy-pasted duplicates.
- Why it matters: Duplicate definitions cause regex drift — if the pattern changes, some copies get missed.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: PR feedback item #1. Function was previously duplicated in guided-flow.ts, state.ts, and workspace-index.ts.

### R002 — `parseMilestoneId` field named `suffix` not `prefix`
- Class: quality-attribute
- Status: active
- Description: The `parseMilestoneId` return type must use `suffix` (not `prefix`) for the 6-char random trailing string. All callers, tests, and comments must use the correct name.
- Why it matters: Misleading field names cause bugs when other contributors use the API.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: PR feedback item #2. The random chars trail the sequential number, making them a suffix.

### R003 — Test assertion helpers use shared `test-helpers.ts`
- Class: quality-attribute
- Status: active
- Description: All GSD test files must import assertion helpers (`assertEq`, `assertTrue`, `assertMatch`) from `tests/test-helpers.ts` via `createTestContext()`. No inline assertion function definitions in test files.
- Why it matters: Duplicated test helpers are maintenance burden and can drift in behavior.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: PR feedback item #3. Helpers were previously copy-pasted across 3+ test files.

### R004 — No "valueable" typo in README
- Class: quality-attribute
- Status: active
- Description: The README must spell "valuable" correctly. No instances of "valueable" anywhere in the codebase.
- Why it matters: Typos in the project README look unprofessional for an OSS PR.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: PR feedback item #4.

### R005 — All existing tests pass after changes
- Class: quality-attribute
- Status: active
- Description: The full GSD test suite must pass with zero failures after the PR quality changes.
- Why it matters: 43 files were changed — regressions are the primary risk.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: Tests use both `npx tsx` runners and `npx vitest run`.

### R006 — Changes follow codebase import/naming conventions
- Class: quality-attribute
- Status: active
- Description: All changed files must follow the codebase's established patterns: import style (`.js` extensions for local imports), naming conventions, file organization, and comment style.
- Why it matters: Inconsistent style in a PR signals carelessness to upstream reviewers.
- Source: user
- Primary owning slice: M003/S01
- Supporting slices: none
- Validation: mapped
- Notes: Convention check across all 43 changed files.

## Validated

(none yet)

## Deferred

(none)

## Out of Scope

(none)

## Traceability

| ID | Class | Status | Primary owner | Supporting | Proof |
|---|---|---|---|---|---|
| R001 | quality-attribute | active | M003/S01 | none | mapped |
| R002 | quality-attribute | active | M003/S01 | none | mapped |
| R003 | quality-attribute | active | M003/S01 | none | mapped |
| R004 | quality-attribute | active | M003/S01 | none | mapped |
| R005 | quality-attribute | active | M003/S01 | none | mapped |
| R006 | quality-attribute | active | M003/S01 | none | mapped |

## Coverage Summary

- Active requirements: 6
- Mapped to slices: 6
- Validated: 0
- Unmapped active requirements: 0
