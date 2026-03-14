# Requirements

This file is the explicit capability and coverage contract for the project.

## Active

(none)

## Validated

### R001 — Welcome message on session start
- Class: primary-user-loop
- Status: validated
- Description: GSD displays "Hello you beautiful person!" when a session starts, visible in the TUI header area
- Why it matters: Friendly greeting sets the tone for the session
- Source: user
- Primary owning slice: M003-alcfdr/S01
- Supporting slices: none
- Validation: source grep confirms string at line 198; build passes clean; UAT written
- Notes: Message should appear on every session start, not just first launch

## Deferred

(none)

## Out of Scope

(none)

## Traceability

| ID | Class | Status | Primary owner | Supporting | Proof |
|---|---|---|---|---|---|
| R001 | primary-user-loop | validated | M003-alcfdr/S01 | none | source grep + build pass + UAT |

## Coverage Summary

- Active requirements: 0
- Mapped to slices: 1
- Validated: 1
- Unmapped active requirements: 0
