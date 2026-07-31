---
schemaVersion: 1
id: frontend-state-any-typing
priority: P2
category: architecture
status: fixed
title: AgentState types resources/logs as any[] while concrete types exist
summary: The frontend AgentState declared resources and logs as any[], erasing type safety across the whole cross-process state contract even though Resource was defined in the same file and a Log shape existed on the backend; both are now concretely typed
fix: Type resources as Resource[] and add a Log type for logs in types.ts (~5 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, components]
cites:
  - src/lib/types.ts:23 :: resources: Resource[]
  - src/lib/types.ts:24 :: logs: Log[]
  - src/lib/types.ts:14 :: export type Log
found: 2026-07-20
fixed: 2026-07-30
type: Improvement
description: "The frontend AgentState declared resources and logs as any[], erasing type safety across the whole cross-process state contract even though Resource was defined in the same file and a Log shape existed on the backend; both are now concretely typed"
tags: [architecture, P2]
timestamp: 2026-07-20
---
`AgentState` is the frontend half of the three-place cross-process shape
([agent-state-shape-contract](/brain/rules/agent-state-shape-contract.md)). `resources` and
`logs` were typed `any[]`, so every consumer lost type checking on the two list fields that
drive the canvas cards and the Progress step list — a rename or field typo on the backend
(`state.py` `Resource`/`Log`) could not be caught on the frontend at compile time. `Resource`
was already defined one block up in the same file, and the backend had a matching `Log`
TypedDict. The brain notes the looseness was "deliberate" — but loose typing is exactly what
lets a silent shape drift render nothing, which is why it was worth tightening.

## Resolution (2026-07-30)

`src/lib/types.ts` now declares `resources: Resource[]` (`:23`) and `logs: Log[]` (`:24`),
with `Log = { message: string; done: boolean }` added as a named export at `:14` — matching
the backend `Log` TypedDict field-for-field. `Resource` was reused as-is. Verified against
current source this pass.

**The old citations are gone on purpose.** This row previously cited
`src/lib/types.ts:11 :: any[]` and `:12 :: any[]`; neither token exists any more, and line 11
is now part of the unrelated `Citation` interface — a cite that still resolved by line number
but pointed at the wrong thing. The cites above point at the fix instead, so the row stays
checkable rather than decorative.

One consequence worth knowing before touching this file: the types are now load-bearing for
`createInitialAgentState` (`types.ts:32`, see
[coagent-initialstate-divergence](/improve/improvements/coagent-initialstate-divergence.md)) —
widening a field back to `any` would silently switch the seed's compile-time check off too.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/lib/types.ts:23](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/types.ts#L23) — `resources: Resource[]`
[2] [src/lib/types.ts:24](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/types.ts#L24) — `logs: Log[]`
[3] [src/lib/types.ts:14](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/types.ts#L14) — `export type Log`

<!-- okf:citations:end -->
