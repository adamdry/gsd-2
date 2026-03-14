# Project

## What This Is

GSD (Get Stuff Done) is a CLI coding agent harness — an extension-based system for structured software delivery with milestone/slice/task workflow management, git integration, and AI-assisted development. Built in TypeScript on Node.js.

## Core Value

Structured, agent-executable project management with milestone/slice/task decomposition and automated git workflow.

## Current State

- M001-us7ooo delivered collision-resistant unique milestone IDs with an opt-in preference toggle and regex hardening across 12+ code sites.
- M002-fhm3kr swapped the unreleased unique ID format from `M-{rand6}-{seq}` to `M{seq}-{rand6}` (e.g. `M001-abc123`). Clean replacement — all production code, tests, and docs updated. 206 assertions pass across 3 test files. Zero old-format traces remain.

## Architecture / Key Patterns

- Extension-based architecture: GSD lives in `src/resources/extensions/gsd/`
- Milestone ID primitives in `guided-flow.ts`: `MILESTONE_ID_RE`, `extractMilestoneSeq`, `parseMilestoneId`, `milestoneIdSort`, `generateMilestonePrefix`, `nextMilestoneId`
- Unique milestone IDs generate `M{seq}-{rand6}` format (e.g. `M001-abc123`) when `unique_milestone_ids: true`; defaults to plain sequential `M001`
- Preferences system in `preferences.ts`: interface, validation, merge, serialization
- Wizard UI in `commands.ts`: `handlePrefsWizard()`
- Tests use custom assertion runners (not vitest `describe`/`it`), run via `npx tsx`
- Regex-hardening test uses vitest wrapper, run via `npx vitest run`
- Logo rendering: `src/logo.ts` exports raw lines and `renderLogo()`, called from `src/loader.ts`, `src/onboarding.ts`, and `src/resources/extensions/gsd/index.ts`

## Capability Contract

See `.gsd/REQUIREMENTS.md` for the explicit capability contract, requirement status, and coverage mapping.

## Milestone Sequence

- [x] M001-us7ooo: Unique Milestone IDs — collision-resistant ID generation with opt-in preference and regex hardening
- [x] M002-fhm3kr: Unique Milestone ID Format Swap — changed unreleased format from `M-{rand6}-{seq}` to `M{seq}-{rand6}`
