---
schemaVersion: 1
id: verifiable-report-initialstate
plan: verifiable-report
parent: verifiable-report
kind: task
title: Unify the two useCoAgent initialState seeds and include the citations default
description: >-
  While-you're-here fix folded in from the open ledger item
  coagent-initialstate-divergence: Main.tsx and ResearchCanvas.tsx each seed a
  separate initialState for the same agent. Adding citations is exactly the
  moment either seed could drift again — unify them into one shared constant
  instead of adding a third divergent copy.
approach: >-
  Extract one shared initialState constant (including citations: []) used by
  both useCoAgent call sites in Main.tsx and ResearchCanvas.tsx.
status: done
priority: 3
---

## Log

- **2026-07-24** — Added `createInitialAgentState(model)` factory to
  `src/lib/types.ts` (single source of truth, incl. `citations: []`); both
  `Main.tsx` and `ResearchCanvas.tsx` now call it instead of seeding divergent
  shapes (`ResearchCanvas` previously seeded only `{ model }`). Commit
  `e0c9058` `[verifiable-report-initialstate] refactor: unify initialState
  seed`. prism independently traced render guards under old vs. new seed and
  confirmed identical output (no behavior change), and confirmed `citations`
  isn't read by any render path so it can't introduce undefined-field issues.
  `report_improvement_status(coagent-initialstate-divergence, fixed)` called
  and confirmed via the platform's `reported` overlay; local ledger file
  frontmatter updated to `status: fixed`. Done-check: prism PASS.

## Done-check

- One shared `initialState` constant (module-level or a small shared module)
  is used by both `useCoAgent` hooks in `Main.tsx` and `ResearchCanvas.tsx`,
  including a `citations: []` default.
- Manual check: loading the app with no prior agent run still renders cleanly
  (no undefined-field errors) at both call sites.
- `report_improvement_status` is called for `coagent-initialstate-divergence`
  at the fix boundary.
