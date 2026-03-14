# S01: PR Quality Audit — UAT

**Milestone:** M003-8m2o9o
**Written:** 2026-03-14

## UAT Type

- UAT mode: artifact-driven
- Why this mode is sufficient: All 6 requirements are mechanically verifiable via grep searches and test suite execution — no runtime behavior or UI to inspect

## Preconditions

- Working directory is the project root (`gsd-2/`)
- Node.js and npm are available
- `ripgrep` (`rg`) is installed

## Smoke Test

Run `npm test` — should exit 0 with all tests passing.

## Test Cases

### 1. findMilestoneIds defined exactly once

1. Run: `rg 'function findMilestoneIds' src/resources/extensions/gsd/ --type ts`
2. **Expected:** Exactly 1 match in `src/resources/extensions/gsd/guided-flow.ts`
3. Run: `rg 'findMilestoneIds' src/resources/extensions/gsd/state.ts src/resources/extensions/gsd/files.ts src/resources/extensions/gsd/workspace-index.ts`
4. **Expected:** Each file has an import line referencing `./guided-flow.js`, no local `function findMilestoneIds` definitions

### 2. parseMilestoneId uses suffix, not prefix

1. Run: `rg '\.prefix' src/resources/extensions/gsd/ --type ts`
2. **Expected:** Zero matches (exit code 1)
3. Run: `rg 'suffix' src/resources/extensions/gsd/guided-flow.ts`
4. **Expected:** At least 1 match showing `suffix` in the return type or destructuring

### 3. No inline assertion helpers outside test-helpers.ts

1. Run: `rg 'function assert(Eq|True|Match)' src/resources/extensions/gsd/tests/ --type ts`
2. **Expected:** All matches are in `src/resources/extensions/gsd/tests/test-helpers.ts` only — no matches in any other test file

### 4. No "valueable" typo anywhere

1. Run: `rg -i 'valueable' .`
2. **Expected:** Zero matches (exit code 1)

### 5. Full test suite passes

1. Run: `npm test`
2. **Expected:** Exit code 0, unit tests report 0 failures, integration tests report 0 failures

### 6. Import conventions consistent

1. Open `src/resources/extensions/gsd/state.ts` and check import of `findMilestoneIds`
2. **Expected:** Import uses `'./guided-flow.js'` (`.js` extension for local imports)
3. Open `src/resources/extensions/gsd/files.ts` and check import of `findMilestoneIds`
4. **Expected:** Same `.js` extension pattern

## Edge Cases

### grep false positives in non-ts files

1. Run: `rg 'function findMilestoneIds' src/resources/extensions/gsd/` (without `--type ts`)
2. **Expected:** No additional matches in `.js`, `.json`, or other file types

### prefix in unrelated contexts

1. Run: `rg 'prefix' src/resources/extensions/gsd/ --type ts` (without the dot)
2. **Expected:** Any matches are for unrelated variables (e.g. `generateMilestonePrefix`) — none reference `.prefix` as a property access on parseMilestoneId results

## Failure Signals

- Any `rg` command returning unexpected matches indicates a stale reference was missed
- `npm test` exit code non-zero indicates a regression in the 43 changed files
- Import using `./guided-flow.ts` instead of `./guided-flow.js` would indicate a convention break

## Requirements Proved By This UAT

- R001 — test case 1 proves single definition + correct imports
- R002 — test case 2 proves suffix naming + no prefix references
- R003 — test case 3 proves shared test helpers, no inline copies
- R004 — test case 4 proves zero typo instances
- R005 — test case 5 proves full test suite passes
- R006 — test case 6 proves import convention consistency

## Not Proven By This UAT

- Runtime behavior of the GSD extension (not in scope — this is a static quality audit)
- Whether the PR meets all upstream maintainer preferences beyond the 4 explicit feedback items

## Notes for Tester

- Mixed `.js`/`.ts` import extensions are a pre-existing codebase pattern, not a defect. The project uses TypeScript with Node.js module resolution that requires `.js` extensions in imports.
- All test cases are deterministic grep/test commands — results should be identical across runs.
