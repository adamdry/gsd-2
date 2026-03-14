# S01: Format swap — production code, tests, and docs — Research

**Date:** 2026-03-14

## Summary

This is a mechanical format swap across 8 production files, 3 test files, and 2 doc files. The old format `M-{rand6}-{seq}` (e.g. `M-abc123-001`) becomes `M{seq}-{rand6}` (e.g. `M001-d24bw5-abc123`). The old regex pattern `M(?:-[a-z0-9]{6}-)?\d+` becomes `M\d+(?:-[a-z0-9]{6})?`. Every site has been inventoried — 16 production code changes, ~114 test data references, and 2 doc references. All three test suites currently pass (208 total assertions), confirming a clean baseline.

The main complexity is in `parseMilestoneId` — its regex capture groups swap position (prefix moves from group 1 to group 2 in the old code, but now the seq digits are always right after `M` and the prefix trails). The `nextMilestoneId` format string flips from `` `M-${prefix}-${seq}` `` to `` `M${seq}-${prefix}` ``.

## Recommendation

Single-pass top-down approach: change the 5 core primitives in `guided-flow.ts` first (MILESTONE_ID_RE, extractMilestoneSeq, parseMilestoneId regex, findMilestoneIds, nextMilestoneId format string), then do a global find-replace of the regex pattern across the other 7 production files, then update test data across all 3 test files, then update 2 doc files. Verify with test runs + grep for zero old-format traces.

## Don't Hand-Roll

No external solutions needed — this is purely internal regex/string work.

## Existing Code and Patterns

### Production files (8 files, 16 change sites)

- `guided-flow.ts` — **Core ID primitives.** 6 sites:
  - L107: `findMilestoneIds` dir scan regex → `M\d+(?:-[a-z0-9]{6})?`
  - L118: comment describing formats → update example
  - L119: `MILESTONE_ID_RE` → `/^M\d{3}(?:-[a-z0-9]{6})?$/`
  - L123: `extractMilestoneSeq` regex → `/^M(\d{3})(?:-[a-z0-9]{6})?$/` (digits now in group 1)
  - L129: `parseMilestoneId` regex → `/^M(\d{3})(?:-([a-z0-9]{6}))?$/` (seq=group1, prefix=group2)
  - L164: `nextMilestoneId` format string → `` `M${seq}-${generateMilestonePrefix()}` ``

- `state.ts` — 2 sites:
  - L66: `findMilestoneIds` dir scan regex
  - L171: title-stripping regex

- `workspace-index.ts` — 2 sites:
  - L68: `findMilestoneIds` dir scan regex
  - L79: title-stripping regex

- `files.ts` — 2 sites:
  - L759: JSDoc comment with pattern description
  - L769: `findMilestoneIds` dir scan regex

- `dispatch-guard.ts` — 1 site:
  - L42: dir scan regex

- `worktree.ts` — 1 site:
  - L98: `SLICE_BRANCH_RE` — milestone capture group within branch pattern

- `worktree-command.ts` — 1 site:
  - L320: milestone directory filter regex

- `index.ts` — 3 sites:
  - L63: `MILESTONE_CONTEXT_RE`
  - L484: `executeMatch` prompt regex
  - L490: `resumeMatch` prompt regex

### Regex pattern mapping

All dir-scan / title-strip / detection regexes follow the same transformation:

| Use | Old pattern | New pattern |
|-----|------------|------------|
| Dir scan (capturing) | `/^(M(?:-[a-z0-9]{6}-)?\d+)/` | `/^(M\d+(?:-[a-z0-9]{6})?)/` |
| Title strip | `/^M(?:-[a-z0-9]{6}-)?\d+[^:]*:\s*/` | `/^M\d+(?:-[a-z0-9]{6})?[^:]*:\s*/` |
| MILESTONE_ID_RE (anchored) | `/^M(?:-[a-z0-9]{6}-)?\d{3}$/` | `/^M\d{3}(?:-[a-z0-9]{6})?$/` |
| Context file match | `/M(?:-[a-z0-9]{6}-)?\d+-CONTEXT\.md$/` | `/M\d+(?:-[a-z0-9]{6})?-CONTEXT\.md$/` |
| Branch pattern | `(M(?:-[a-z0-9]{6}-)?\d+)` | `(M\d+(?:-[a-z0-9]{6})?)` |
| Prompt dispatch | `(M(?:-[a-z0-9]{6}-)?\d+)` | `(M\d+(?:-[a-z0-9]{6})?)` |
| Milestone detect | `/^M(?:-[a-z0-9]{6}-)?\d+/` | `/^M\d+(?:-[a-z0-9]{6})?/` |

### Test files (3 files, ~114 old-format references)

- `unique-milestone-ids.test.ts` — 15 refs. Test data uses `M-abc123-001`, `M-z9a8b7-042`, `M-def456-002`, `M-xyz999-005`. Key changes:
  - MILESTONE_ID_RE match/reject cases flip (e.g. `M-abc123-001` → `M001-d24bw5-abc123`)
  - `extractMilestoneSeq` test data
  - `parseMilestoneId` group expectations — prefix/num swap in structured output
  - `milestoneIdSort` test arrays
  - `nextMilestoneId` assertions: `startsWith('M-')` → different check, `endsWith('-001')` → different check
  - `maxMilestoneNum` test data

- `regex-hardening.test.ts` — 30 refs. Contains duplicated regex patterns in comments and local const declarations for each test group. All pattern strings and test data need updating. The structure is: comment with pattern → local regex const → match/reject test cases.

- `integration-mixed-milestones.test.ts` — 69 refs. Heavy use of `M-abc123-001`, `M-abc123-002` in fixture creation, assertions, and string messages. Systematic replacement of `M-abc123-NNN` → `MNNN-abc123`.

### Doc files (2 files, 1 reference each)

- `docs/preferences-reference.md` L110: `M-{rand6}-{seq}` format description and `M-eh88as-001` example
- `prompts/system.md` L53: `M-{rand6}-{seq}/` format description and `M-eh88as-001/` example

### Template file — no changes needed

- `templates/preferences.md` — confirmed no format-specific content

## Constraints

- Must use `crypto.randomInt()` for random generation (D002 — unchanged)
- Must preserve `milestoneIdSort` comparator behavior — sort by sequential number (unchanged, but test data updates needed)
- `parseMilestoneId` return type `{ prefix?: string; num: number }` stays the same — only the regex capture group indices change
- `MILESTONE_ID_RE` must remain anchored (`^...$`) for validation use
- Dir-scan regexes must remain unanchored at end (directories may have trailing content)
- The `extractMilestoneSeq` and `parseMilestoneId` regexes use `\d{3}` (fixed 3 digits), while dir-scan uses `\d+` (variable length) — this distinction must be preserved
- `SLICE_BRANCH_RE` in `worktree.ts` embeds the milestone pattern inside a larger regex — careful with group numbering

## Common Pitfalls

- **`parseMilestoneId` capture group swap** — Old regex: `M(?:-([a-z0-9]{6})-)?(\d{3})` has prefix=m[1], num=m[2]. New regex: `M(\d{3})(?:-([a-z0-9]{6}))?` has num=m[1], prefix=m[2]. The return statement must swap which group maps to which field.
- **`extractMilestoneSeq` group change** — Old: `(\d{3})` was group 1 (only capturing group). New: `(\d{3})` is still group 1 (prefix group is non-capturing or group 2), so `m[1]` stays correct — but the regex structure changes so verify the group index.
- **`nextMilestoneId` assertion style in tests** — Old tests check `startsWith('M-')` and `endsWith('-001')`. New format: should check `startsWith('M001-d24bw5-')` pattern and use regex match. The seq is now at position 1-3, prefix at position 5-10.
- **`SLICE_BRANCH_RE` group numbering** — This regex has 3 capture groups. The milestone group is group 2. Changing the inner pattern structure must not break the group index that callers rely on.
- **Blocker message strings in integration tests** — Tests assert on error messages like `earlier slice M-abc123-002/S01 is not complete`. These messages are generated by production code using the actual milestone ID, so they'll naturally use the new format — but the test regex/string assertions need updating to match.

## Open Risks

- **Zero** — this is a well-understood mechanical replacement on unreleased code with comprehensive test coverage (208 assertions). All change sites are inventoried. The new pattern `M\d+(?:-[a-z0-9]{6})?` is structurally simpler than the old one.

## Skills Discovered

No relevant skills needed — this is a mechanical regex/string replacement task, not a framework or technology integration.

## Sources

- All findings from direct codebase exploration (grep, rg, file reads)
- D007 and D008 in DECISIONS.md define the new format and regex pattern
