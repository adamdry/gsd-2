---
id: M001
provides:
  - "MILESTONE_ID_RE, extractMilestoneSeq, parseMilestoneId, milestoneIdSort, generateMilestonePrefix, nextMilestoneId — ID primitives for both M001 and M-{rand6}-{seq} formats"
  - "GSDPreferences.unique_milestone_ids — opt-in preference plumbed through interface, validation, merge, serialization, and wizard"
  - "All 12+ regex/parser sites widened to accept both milestone ID formats — zero M\\d+-only patterns remain in production code"
  - "dispatch-guard.ts refactored from parseInt(id.slice(1)) to directory-scanning approach via readdirSync + extractMilestoneSeq"
  - "Integration test suite proving deriveState, indexWorkspace, inlinePriorMilestoneSummary, getPriorSliceCompletionBlocker, and branch operations with mixed-format milestone directories"
key_decisions:
  - "D001: Milestone ID format M-{rand6}-{seq} with opt-in preference — collision-resistant, backwards compatible"
  - "D002: crypto.randomInt() for random generation — no Math.random() fallback"
  - "D003: Invalid ID handling returns 0/{num:0} instead of throwing — prevents NaN propagation"
  - "D004: Dispatch guard uses directory scanning via readdirSync + milestonesDir() — replaces parseInt(id.slice(1))"
  - "D005: Regex pattern M(?:-[a-z0-9]{6}-)?\\d+ — single pattern matches both formats"
patterns_established:
  - "ID format abstraction via parseMilestoneId/extractMilestoneSeq — all code paths use these instead of direct string manipulation"
  - "milestoneIdSort comparator — all milestone array sorting uses numeric extraction, not lexicographic order"
  - "Preference-driven feature toggle — unique_milestone_ids flows from preferences.md through validation/merge to generation call sites"
observability_surfaces:
  - "/gsd prefs status — shows effective preferences including unique_milestone_ids"
  - "/gsd prefs wizard — interactive toggle for unique_milestone_ids"
requirement_outcomes: []
duration: "4 slices across M001"
verification_result: passed
completed_at: 2026-03-14
---

# M001: Unique Milestone IDs

**Collision-resistant milestone ID generation (`M-{rand6}-{seq}`) with opt-in preference, full backwards compatibility for legacy `M001` format, and comprehensive regex/parser hardening across all 12+ code sites.**

## What Happened

The milestone delivered a complete ID format abstraction layer for GSD milestones. S01 built the foundation — six ID primitives (`MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId`, `milestoneIdSort`, `generateMilestonePrefix`, `nextMilestoneId`) plus the `unique_milestone_ids` preference field plumbed through the preferences interface, validation, merge, and serialization. S02 then consumed these primitives to widen all 12 regex/parser sites across 8 production files (`state.ts`, `workspace-index.ts`, `files.ts`, `worktree.ts`, `worktree-command.ts`, `index.ts`, `dispatch-guard.ts`, `guided-flow.ts`), replacing every `M\d+`-only pattern with the universal `M(?:-[a-z0-9]{6}-)?\d+` pattern and swapping all 4+ bare `.sort()` calls on milestone arrays with `milestoneIdSort`. The dispatch guard was refactored from the fragile `parseInt(id.slice(1))` approach to directory scanning via `readdirSync`. S03 added the UX surface — a wizard toggle in `/gsd prefs wizard` and documentation in `preferences-reference.md`, `templates/preferences.md`, and `system.md`. S04 sealed the milestone with 62 integration test assertions proving real filesystem and git operations work correctly with new-format and mixed-format milestone directories across `deriveState`, `indexWorkspace`, `inlinePriorMilestoneSummary`, `getPriorSliceCompletionBlocker`, and branch operations.

## Cross-Slice Verification

**Success criterion: User can enable `unique_milestone_ids: true` and all new milestones receive `M-{rand6}-{seq}` format IDs**
- ✅ Verified: `nextMilestoneId([], true)` generates `M-{rand6}-001` format. Unit test section (f) covers this with 63 total assertions passing.

**Success criterion: Existing `M001`-format milestones continue to work identically**
- ✅ Verified: All existing test suites pass without regression. `derive-state.test.ts` (106 passed), `workspace-index.test.ts` (10 passed), `worktree.test.ts` (97 passed), `dispatch-guard.test.ts` (4 passed).

**Success criterion: Mixed-format milestone directories sort, parse, and display correctly**
- ✅ Verified: `milestoneIdSort` comparator tested in `unique-milestone-ids.test.ts` section (d). Integration tests in S04 (62 assertions) prove mixed `M001` + `M-abc123-001` directories work through `deriveState` and `indexWorkspace`.

**Success criterion: `/gsd prefs wizard` includes the unique milestone IDs toggle**
- ✅ Verified: `commands.ts` lines 397-403 implement the wizard prompt with current value display and `(keep current)` option.

**Success criterion: All documentation describes the new format**
- ✅ Verified: `preferences-reference.md` line 110, `templates/preferences.md` line 18, `system.md` line 53 all reference `unique_milestone_ids`.

**Success criterion: All regex/parser sites accept both ID formats — zero `M\d+`-only patterns remain**
- ✅ Verified: `grep -rn 'M\\d+' src/resources/extensions/gsd/{state,workspace-index,files,worktree,worktree-command,index,dispatch-guard}.ts` returns zero hits. `regex-hardening.test.ts` covers all 12 sites with 83 assertions.

**Success criterion: Integration tests prove real filesystem and git operations work with new-format milestone directories**
- ✅ Verified: `integration-mixed-milestones.test.ts` passes with 62 assertions across 6 groups covering `deriveState`, `indexWorkspace`, `inlinePriorMilestoneSummary`, `getPriorSliceCompletionBlocker`, and branch operations.

**Definition of Done verification:**
- ✅ All ID primitives exported and tested (63 unit test assertions)
- ✅ `GSDPreferences.unique_milestone_ids` plumbed through interface (line 86), validation (line 656), merge (line 576), and serialization
- ✅ Zero `M\d+`-only patterns in production code (grep confirms)
- ✅ 5 `.sort(milestoneIdSort)` calls across state.ts, workspace-index.ts, files.ts, dispatch-guard.ts, guided-flow.ts
- ✅ `dispatch-guard.ts` uses `readdirSync` + `extractMilestoneSeq` (lines 2-5, 33, 39, 47)
- ✅ Wizard toggle (commands.ts:397-403), preferences reference, template, and system prompt all document `unique_milestone_ids`
- ✅ Integration tests pass with 62 assertions
- ✅ All existing GSD test suites pass without regression

## Requirement Changes

- No requirements document exists (`legacy compatibility mode` per D006). No requirement transitions to report.

## Forward Intelligence

### What the next milestone should know
- The `M(?:-[a-z0-9]{6}-)?\d+` pattern is the canonical regex for milestone IDs — any new regex site touching milestone IDs must use this pattern, not `M\d+`
- `milestoneIdSort` must be used for any new code that sorts milestone ID arrays — bare `.sort()` produces wrong lexicographic order with mixed formats
- The `unique_milestone_ids` preference is already in production and can generate real `M-{rand6}-{seq}` directories — the project itself used this to create `M-w2lhcz-002`

### What's fragile
- Regex sprawl detection — if a new file adds milestone ID matching without using `MILESTONE_ID_RE` or the widened pattern, it will silently break for new-format IDs. A `grep -rn 'M\\d+' src/resources/extensions/gsd/` audit should be run periodically.
- The `extractMilestoneSeq` function returns 0 for non-matching inputs rather than throwing — callers must handle the 0 case explicitly.

### Authoritative diagnostics
- `grep -rn 'M\\d+' src/resources/extensions/gsd/{state,workspace-index,files,worktree,worktree-command,index,dispatch-guard}.ts` — must return zero hits; any output means a regression
- `npx tsx src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — 63 assertions covering all ID primitives
- `npx tsx src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — 62 integration assertions with real filesystem

### What assumptions changed
- Originally assumed 12 regex sites needed widening — actual count was 12+ (some files had multiple patterns). All were found and fixed.
- The dispatch guard refactoring (D004) was more impactful than expected — it eliminated the entire `parseInt(id.slice(1))` approach which was fundamentally incompatible with the new format.

## Files Created/Modified

- `src/resources/extensions/gsd/guided-flow.ts` — ID primitives (MILESTONE_ID_RE, extractMilestoneSeq, parseMilestoneId, milestoneIdSort, generateMilestonePrefix, nextMilestoneId, maxMilestoneNum) and call-site wiring
- `src/resources/extensions/gsd/preferences.ts` — unique_milestone_ids field in GSDPreferences interface, validation, and merge
- `src/resources/extensions/gsd/state.ts` — widened findMilestoneIds regex, milestoneIdSort import
- `src/resources/extensions/gsd/workspace-index.ts` — widened findMilestoneIds regex, milestoneIdSort import
- `src/resources/extensions/gsd/files.ts` — widened regex for findMilestoneIds and title stripping
- `src/resources/extensions/gsd/dispatch-guard.ts` — refactored to directory scanning with extractMilestoneSeq
- `src/resources/extensions/gsd/worktree.ts` — widened SLICE_BRANCH_RE to capture new-format milestone IDs
- `src/resources/extensions/gsd/worktree-command.ts` — widened branch matching regex
- `src/resources/extensions/gsd/index.ts` — widened hasExistingMilestones and MILESTONE_CONTEXT_RE
- `src/resources/extensions/gsd/commands.ts` — wizard toggle for unique_milestone_ids in /gsd prefs wizard
- `src/resources/extensions/gsd/docs/preferences-reference.md` — unique_milestone_ids field documentation
- `src/resources/extensions/gsd/templates/preferences.md` — unique_milestone_ids field in template
- `src/resources/extensions/gsd/prompts/system.md` — milestone directory format documentation for new format
- `src/resources/extensions/gsd/tests/unique-milestone-ids.test.ts` — 63 unit test assertions for ID primitives
- `src/resources/extensions/gsd/tests/regex-hardening.test.ts` — 83 assertions for all 12 regex/parser sites
- `src/resources/extensions/gsd/tests/integration-mixed-milestones.test.ts` — 62 integration assertions with real filesystem and git
