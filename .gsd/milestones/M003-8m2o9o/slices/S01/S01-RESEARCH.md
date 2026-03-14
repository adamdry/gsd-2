# M003-8m2o9o/S01 — Research

**Date:** 2026-03-14

## Summary

All 4 PR feedback fixes appear correctly implemented. The codebase is clean against each requirement, with one minor edge case worth noting.

R001 (`findMilestoneIds` dedup): The function is defined exactly once in `guided-flow.ts` and imported by all 3 callers (`state.ts`, `files.ts`, `workspace-index.ts`). The duplicates were removed — `state.ts` lost its inline copy and `files.ts` replaced its inline scanning logic with a call to the shared function.

R002 (`suffix` not `prefix`): `parseMilestoneId` in `guided-flow.ts` returns `{ suffix?: string; num: number }`. Zero occurrences of `.prefix` anywhere in the GSD extension directory. Tests correctly reference `suffix` throughout.

R003 (shared test helpers): All 32 test files import `createTestContext()` from `test-helpers.ts`. No inline definitions of `assertEq`, `assertTrue`, or `assertMatch` outside `test-helpers.ts`. One domain-specific `assertClose` exists in `metrics.test.ts` for floating-point tolerance comparison — this is a specialized helper not covered by R003's scope (which names the 3 standard helpers).

R004 ("valueable" typo): Zero matches for "valueable" anywhere in the codebase.

## Recommendation

Proceed directly to planning a single verification task. This is a mechanical audit — run greps for each requirement, run the test suite, and document the results. No code changes needed unless a defect surfaces.

The verification should be ordered: stale-pattern greps first (fast, high confidence), then full test suite run (slower, catches regressions), then import convention spot-check.

## Don't Hand-Roll

| Problem | Existing Solution | Why Use It |
|---------|------------------|------------|
| Test running | `npm test` (`node --test` runner) | Already configured in package.json |
| Pattern searching | `rg` with `--type ts` | Precise, fast, handles the 43-file scope easily |

## Existing Code and Patterns

- `src/resources/extensions/gsd/guided-flow.ts` — canonical home of `findMilestoneIds`, `parseMilestoneId`, `generateMilestoneSuffix`, `milestoneIdSort`
- `src/resources/extensions/gsd/state.ts` — imports `findMilestoneIds` from guided-flow; duplicate was removed here
- `src/resources/extensions/gsd/files.ts` — imports `findMilestoneIds` from guided-flow; inline scanning logic was replaced
- `src/resources/extensions/gsd/workspace-index.ts` — imports `findMilestoneIds` from guided-flow
- `src/resources/extensions/gsd/tests/test-helpers.ts` — shared `createTestContext()` factory providing `assertEq`, `assertTrue`, `assertMatch`, `assertNoMatch` with isolated pass/fail counters
- `src/resources/extensions/gsd/tests/metrics.test.ts` — contains domain-specific `assertClose()` for floating-point tolerance (not a standard helper)

## Constraints

- Tests use `node --test` runner (not vitest despite some comments referencing it). Command: `npm test` which runs `test:unit && test:integration`.
- No vitest config exists in the repo. The `test-helpers.ts` comment about "vitest workers" is outdated documentation.
- Import extensions are mixed in the codebase: `.js` in some production files (`commands.ts`, `doctor.ts`, `auto.ts`), `.ts` in most others. The PR's `guided-flow.js` imports match the pre-existing extension used in the same import blocks — not a PR regression.
- The `--experimental-strip-types` flag is used by the test runner, indicating Node's built-in TypeScript support.

## Common Pitfalls

- **Mixed import extensions (.js vs .ts)** — The codebase uses both conventions. PR imports from `guided-flow.js` match the pre-existing pattern in the same files (e.g., `files.ts` already imported `paths.js`). Don't flag this as a PR issue; it's pre-existing.
- **assertClose is not a violation of R003** — It's a domain-specific tolerance helper, not one of the 3 standard assertion functions. R003 specifically names `assertEq`, `assertTrue`, `assertMatch`.

## Open Risks

- Test suite hasn't been run yet — regressions are the primary unknown. All static checks pass, but import/runtime errors would only surface in the test run.
- No vitest runner exists despite milestone context mentioning it. Only `npm test` (node --test) is available.

## Skills Discovered

| Technology | Skill | Status |
|------------|-------|--------|
| Node.js test runner | none | n/a — built-in, no skill needed |
| TypeScript ESM | none | n/a — standard tooling |

## Sources

- All findings from direct codebase inspection via `rg`, `git diff`, and file reads
- `package.json` test scripts for runner configuration
- `git diff ca6d44bd~1..ca6d44bd` for understanding exact PR changes
