---
schemaVersion: 1
id: verifiable-report-frontend-typing
plan: verifiable-report
parent: verifiable-report
kind: task
title: Type resources/logs (and the new citations) with real types instead of any[]
description: >-
  While-you're-here fix folded in from the open ledger item
  frontend-state-any-typing: src/lib/types.ts types resources and logs as
  any[] even though concrete shapes exist on the backend. Doing this alongside
  the citations field addition avoids a second pass over the same file.
approach: >-
  Type resources: Resource[], add a Log type matching Progress.tsx's inline
  shape, type logs: Log[] in src/lib/types.ts.
status: done
priority: 3
---

## Log

- **2026-07-24** — Added `Log` type (`{message: string, done: boolean}`,
  matching `Progress.tsx`'s inline shape) to `src/lib/types.ts`, retyped
  `AgentState.resources` → `Resource[]` and `AgentState.logs` → `Log[]`.
  Commit `9f6ba83` `[verifiable-report-frontend-typing] fix: type AgentState
  resources/logs`. prism traced every reader/writer (add/remove/update
  resource logic, Progress's log rendering) and confirmed no site relied on
  `any`-looseness. `report_improvement_status(frontend-state-any-typing,
  fixed)` called and confirmed via the platform's `reported` overlay; local
  ledger file frontmatter updated to `status: fixed` to match. Done-check:
  prism PASS.

## Done-check

- `src/lib/types.ts` `AgentState.resources` is `Resource[]` (not `any[]`), a
  `Log` type is declared (matching `Progress.tsx`'s inline shape), and
  `AgentState.logs` is `Log[]`.
- `npm run build` passes with no new type errors.
- `report_improvement_status` is called for `frontend-state-any-typing` at the
  fix boundary (fixed-in-passing is always reported).
