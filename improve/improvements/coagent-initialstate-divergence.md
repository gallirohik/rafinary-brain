---
schemaVersion: 1
id: coagent-initialstate-divergence
priority: P2
category: architecture
status: fixed
title: Two useCoAgent hooks for the same agent seed different initialState
summary: Main.tsx seeds the full AgentState (model, research_question, resources, report, logs) while ResearchCanvas.tsx seeds only { model } for the same agent name — a silent inconsistency where the initial shape depends on mount order
fix: Share one initialState (extract a constant or seed the full shape in both) so the two hooks agree (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, agent-bridge]
cites:
  - src/app/Main.tsx:12 :: initialState
  - src/components/ResearchCanvas.tsx:24 :: initialState
  - src/lib/types.ts:32 :: createInitialAgentState
found: 2026-07-20
fixed: 2026-07-24
fixed_by: verifiable-report-initialstate
type: Improvement
description: "Main.tsx seeds the full AgentState (model, research_question, resources, report, logs) while ResearchCanvas.tsx seeds only { model } for the same agent name — a silent inconsistency where the initial shape depends on mount order"
timestamp: 2026-07-20
---
Both `Main` and `ResearchCanvas` call `useCoAgent<AgentState>({ name: agent })` against the
**same** coagent ([agent-name-contract](/brain/rules/agent-name-contract.md)), but used to pass
different `initialState`: `Main` seeded all five fields (`model`, `research_question`,
`resources`, `report`, `logs`), while `ResearchCanvas` seeded only `{ model }`. Whichever
hook initialized the shared state first won, so the canvas's starting shape was order-dependent
rather than declared. It worked because `Main` mounts around `ResearchCanvas`, but it was a
latent footgun: reorder the tree and `resources`/`report`/`logs` start `undefined`, which the
render code papers over with `|| []` / `|| ""` guards.

## Resolution (2026-07-24, plan `verifiable-report-initialstate`)

`src/lib/types.ts:32-41` now exports a single `createInitialAgentState(model)` factory — a
factory rather than a constant because `model` is only known at render time from
`useModelSelectorContext`. Both call sites seed from it: `Main.tsx:12` and
`ResearchCanvas.tsx:24`. The seeded shape now includes `citations: []` as well, so all six
fields of [the state contract](/brain/rules/agent-state-shape-contract.md) start declared and
mount order is irrelevant. Verified against current source this pass.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/Main.tsx:12](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/Main.tsx#L12) — `initialState`
[2] [src/components/ResearchCanvas.tsx:24](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/components/ResearchCanvas.tsx#L24) — `initialState`
[3] [src/lib/types.ts:32](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/lib/types.ts#L32) — `createInitialAgentState`

<!-- okf:citations:end -->
