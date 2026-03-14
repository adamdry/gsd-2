# M002: Unique Milestone ID Format Swap

**Vision:** Swap the unreleased unique milestone ID format from `M-{rand6}-{seq}` to `M{seq}-{rand6}` (e.g. `M001-abc123`) with zero traces of the old format remaining. Clean replacement — no migration, no backwards compatibility.

## Success Criteria

- `nextMilestoneId([], true)` generates `M001-{rand6}` format IDs
- All regex/parser sites use `M\d+(?:-[a-z0-9]{6})?` pattern
- Zero traces of old `M(?:-[a-z0-9]{6}-)?\d+` pattern in production code
- Zero traces of old `M-abc123-001` style examples in tests or docs
- All three milestone-related test suites pass with new format data
- All existing GSD test suites pass without regression

## Key Risks / Unknowns

- Low risk — mechanical format swap on unreleased code with comprehensive existing test coverage

## Proof Strategy

- Regex completeness → single slice with grep verification confirming zero old-pattern traces

## Verification Classes

- Contract verification: unit tests (`unique-milestone-ids.test.ts`, `regex-hardening.test.ts`), integration tests (`integration-mixed-milestones.test.ts`), grep for zero old-format patterns
- Integration verification: integration test suite proves `deriveState`, `indexWorkspace`, branch ops work with new format
- Operational verification: none
- UAT / human verification: none needed — format change is fully testable

## Milestone Definition of Done

This milestone is complete only when all are true:

- `nextMilestoneId(milestoneIds, true)` generates `M{seq}-{rand6}` format (e.g. `M001-abc123`)
- `MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId` all handle `M001-abc123` format
- All regex/parser sites across 8 production files use `M\d+(?:-[a-z0-9]{6})?` pattern
- `grep -rn 'M(?:-' src/resources/extensions/gsd/` returns zero hits
- `grep -rn 'M-[a-z0-9]\{6\}-\|M-abc123-\|M-eh88as-\|M-w2lhcz-' src/resources/extensions/gsd/` returns zero hits
- All three test files updated and passing with new-format test data
- Docs (`preferences-reference.md`, `system.md`) updated with new format examples
- All existing GSD test suites pass without regression

## Requirement Coverage

- Covers: N/A (no requirements file — legacy compatibility mode)
- Partially covers: N/A
- Leaves for later: N/A
- Orphan risks: N/A

## Slices

- [x] **S01: Format swap — production code, tests, and docs** `risk:low` `depends:[]`
  > After this: `nextMilestoneId([], true)` returns `M001-{rand6}`, all ~208 test assertions pass with new format, grep confirms zero traces of old `M-{rand6}-{seq}` pattern anywhere in the codebase.

## Boundary Map

### S01 (only slice)

Produces:
- `guided-flow.ts` → updated `MILESTONE_ID_RE` (`/^M\d{3}(?:-[a-z0-9]{6})?$/`), `extractMilestoneSeq`, `parseMilestoneId`, `nextMilestoneId` — all generating/parsing `M{seq}-{rand6}` format
- All 8 production files → regex pattern `M\d+(?:-[a-z0-9]{6})?` replacing `M(?:-[a-z0-9]{6}-)?\d+`
- 3 test files → all test data using `M001-abc123` format instead of `M-abc123-001`
- 2 doc files → updated format descriptions and examples

Consumes:
- nothing (single slice milestone)
