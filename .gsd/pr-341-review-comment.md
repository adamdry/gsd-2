I was doing some inspection to check our PRs would play nicely together and I wanted to flag a subtle interaction with the `.gsd/milestones/` force-add that I noticed while reviewing.

**`continue.md` files would get accidentally committed**

Our gitignore setup (both the selective version we recommend for teams and any blanket `.gsd/` pattern) intentionally excludes session-specific interrupted-work markers:

```
.gsd/milestones/**/continue.md
.gsd/milestones/**/*-CONTINUE.md
```

These are per-developer ephemeral files that shouldn't be shared via git. The issue is in the staging flow with your change applied:

1. `git add -A` — respects `.gitignore`, correctly skips `continue.md` ✅
2. `git add --force -- .gsd/milestones/` — force-adds **everything** under milestones, including `continue.md` and `*-CONTINUE.md` ❌
3. `git reset HEAD -- <RUNTIME_EXCLUSION_PATHS>` — doesn't un-stage them because `continue.md` isn't in `RUNTIME_EXCLUSION_PATHS` ❌
4. `continue.md` gets committed

This applies regardless of whether the user has a blanket `.gsd/` gitignore or the selective one — `git add --force` overrides both.

A couple of possible fixes:

- Add the continue patterns to `RUNTIME_EXCLUSION_PATHS` (or add a dedicated reset step after the force-add)
- Make the force-add more granular — target specific file suffixes (`*-CONTEXT.md`, `*-ROADMAP.md`, `*-PLAN.md`, `*-SUMMARY.md`, `*-UAT.md`, `*-RESEARCH.md`) instead of the whole directory
