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
status: todo
priority: 3
blocked_by: [verifiable-report-state]
---

## Done-check

- `src/lib/types.ts` `AgentState.resources` is `Resource[]` (not `any[]`), a
  `Log` type is declared (matching `Progress.tsx`'s inline shape), and
  `AgentState.logs` is `Log[]`.
- `npm run build` passes with no new type errors.
- `report_improvement_status` is called for `frontend-state-any-typing` at the
  fix boundary (fixed-in-passing is always reported).
