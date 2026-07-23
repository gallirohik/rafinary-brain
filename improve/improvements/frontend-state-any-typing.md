---
schemaVersion: 1
id: frontend-state-any-typing
priority: P2
lens: architecture
status: open
title: AgentState types resources/logs as any[] while concrete types exist
summary: The frontend AgentState declares resources and logs as any[], erasing type safety across the whole cross-process state contract even though Resource is defined in the same file and a Log shape exists on the backend
fix: Type resources as Resource[] and add a Log type for logs in types.ts (~5 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, components]
cites:
  - src/lib/types.ts:11 :: any[]
  - src/lib/types.ts:12 :: any[]
found: 2026-07-20
---
`AgentState` (`types.ts:7-13`) is the frontend half of the three-place cross-process shape
([agent-state-shape-contract](/brain/rules/agent-state-shape-contract.md)). `resources` and
`logs` are typed `any[]` (`types.ts:11-12`), so every consumer loses type checking on the two
list fields that drive the canvas cards and the Progress step list — a rename or field typo on
the backend (`state.py` `Resource`/`Log`, `state.py:10-26`) can't be caught on the frontend at
compile time. `Resource` is already defined one block up in the same file (`types.ts:1-5`), and
the backend has a matching `Log` TypedDict; typing `resources: Resource[]` and adding a `Log`
type restores safety for free. The brain notes this is "deliberately loose" — but the loose typing
is what lets a silent shape drift render nothing, so it is worth tightening while nearby.
`Progress.tsx:5-12` already declares the exact log shape inline, so the type is known.
