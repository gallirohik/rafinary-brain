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
status: todo
priority: 3
blocked_by: [verifiable-report-state]
---

## Done-check

- One shared `initialState` constant (module-level or a small shared module)
  is used by both `useCoAgent` hooks in `Main.tsx` and `ResearchCanvas.tsx`,
  including a `citations: []` default.
- Manual check: loading the app with no prior agent run still renders cleanly
  (no undefined-field errors) at both call sites.
- `report_improvement_status` is called for `coagent-initialstate-divergence`
  at the fix boundary.
