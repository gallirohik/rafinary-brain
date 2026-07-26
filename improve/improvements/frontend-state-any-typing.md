---
schemaVersion: 1
id: frontend-state-any-typing
priority: P2
category: architecture
status: fixed
title: AgentState types resources/logs as any[] while concrete types exist
summary: The frontend AgentState declares resources and logs as any[], erasing type safety across the whole cross-process state contract even though Resource is defined in the same file and a Log shape exists on the backend
fix: Type resources as Resource[] and add a Log type for logs in types.ts (~5 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, components]
cites:
  - src/lib/types.ts:23 :: Resource[]
  - src/lib/types.ts:24 :: Log[]
  - src/lib/types.ts:14 :: Log
found: 2026-07-20
fixed: 2026-07-24
fixed_by: verifiable-report-frontend-typing
---
`AgentState` is the frontend half of the cross-process shape
([agent-state-shape-contract](/brain/rules/agent-state-shape-contract.md)). `resources` and
`logs` were typed `any[]`, so every consumer lost type checking on the two list fields that
drive the canvas cards and the Progress step list — a rename or field typo on the backend
(`state.py` `Resource`/`Log`, `state.py:10-26`) could not be caught on the frontend at compile
time. `Resource` was already defined one block up in the same file, and the backend had a
matching `Log` TypedDict, so the fix was free.

## Resolution (2026-07-24, plan `verifiable-report-frontend-typing`)

`src/lib/types.ts` now declares a `Log` type (`:14-17`, `{ message, done }` — matching
`state.py`'s `Log` TypedDict and the shape `Progress.tsx` renders) and `AgentState` types the
list fields concretely: `resources: Resource[]` (`:23`), `logs: Log[]` (`:24`). A `Citation`
type (`:7-12`) and `citations: Citation[]` (`:25`) landed alongside in the same epic. `any` no
longer appears in `types.ts`. Verified against current source this pass.

