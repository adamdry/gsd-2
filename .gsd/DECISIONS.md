# Decisions Register

<!-- Append-only. Never edit or remove existing rows.
     To reverse a decision, add a new row that supersedes it.
     Read this file at the start of any planning or research phase. -->

| # | When | Scope | Decision | Choice | Rationale | Revisable? |
|---|------|-------|----------|--------|-----------|------------|
| D001 | M001-us7ooo | arch | Milestone ID format | `M-{rand6}-{seq}` with opt-in preference | Collision-resistant for multi-user/multi-project, backwards compatible with sequential `M001` | Yes — if format needs to change |
| D002 | M001-us7ooo/S01 | library | Random generation | `crypto.randomInt()` | Cryptographically sound, no `Math.random()` fallback | No |
| D003 | M001-us7ooo/S01 | convention | Invalid ID handling | Return 0/`{ num: 0 }` instead of throwing | Callers can detect failures without try/catch; prevents NaN propagation | No |
| D004 | M001-us7ooo/S02 | arch | Dispatch guard approach | Directory scanning via `readdirSync` + `milestonesDir()` | Replaces `parseInt(id.slice(1))` which returns NaN for new format | No |
| D005 | M001-us7ooo/S02 | pattern | Regex pattern for both formats | `M(?:-[a-z0-9]{6}-)?\d+` | Single pattern matches `M001` and `M-abc123-001`; optional non-capturing group | No |
| D006 | M001-us7ooo | scope | Requirements tracking | Legacy compatibility mode | No `REQUIREMENTS.md` found at planning time; requirement coverage cannot be mapped | Yes — when REQUIREMENTS.md is created |
| D007 | M002-fhm3kr | arch | Milestone ID format change | `M{seq}-{rand6}` (e.g. `M001-abc123`) supersedes D001's `M-{rand6}-{seq}` | Sequential number leads for natural sort and pattern consistency with plain `M001`; old format never released so no migration needed | No |
| D008 | M002-fhm3kr | pattern | Regex pattern for both formats | `M\d+(?:-[a-z0-9]{6})?` supersedes D005's `M(?:-[a-z0-9]{6}-)?\d+` | Simpler pattern; matches `M001` and `M001-abc123`; sequential digits always at front | No |
| D009 | M003-alcfdr | convention | Welcome message location | `session_start` hook in gsd extension index.ts | Existing header rendering lives there; consistent with branded banner pattern | No |
