---
schemaVersion: 1
id: coagent-initialstate-divergence
priority: P2
category: architecture
status: fixed
title: Two useCoAgent hooks for the same agent seed different initialState
summary: Main.tsx seeded the full AgentState (model, research_question, resources, report, logs) while ResearchCanvas.tsx seeded only { model } for the same agent name — a silent inconsistency where the initial shape depended on mount order; both now call one shared createInitialAgentState factory
fix: Share one initialState (extract a constant or seed the full shape in both) so the two hooks agree (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, agent-bridge]
cites:
  - src/lib/types.ts:32 :: createInitialAgentState
  - src/app/Main.tsx:12 :: createInitialAgentState
  - src/components/ResearchCanvas.tsx:24 :: createInitialAgentState
found: 2026-07-20
fixed: 2026-07-30
type: Improvement
description: "Main.tsx seeded the full AgentState (model, research_question, resources, report, logs) while ResearchCanvas.tsx seeded only { model } for the same agent name — a silent inconsistency where the initial shape depended on mount order; both now call one shared createInitialAgentState factory"
tags: [architecture, P2]
timestamp: 2026-07-20
---
Both `Main` and `ResearchCanvas` call `useCoAgent<AgentState>({ name: agent })` against the
**same** coagent ([agent-name-contract](/brain/rules/agent-name-contract.md)), but passed
different `initialState`: `Main.tsx` seeded all five fields (`model`, `research_question`,
`resources`, `report`, `logs`), while `ResearchCanvas.tsx` seeded only `{ model }`. Whichever
hook initialized the shared state first won, so the canvas's starting shape was order-dependent
rather than declared. It worked because `Main` mounts around `ResearchCanvas`, but it was a
latent footgun: reorder the tree and `resources`/`report`/`logs` start `undefined`, which the
render code papers over with `|| []` / `|| ""` guards. Define the initial state once and use it in
both places so the contract is explicit, not incidental.

## Resolution (2026-07-30)

Closed by a shared factory rather than a duplicated literal. `src/lib/types.ts:32` now exports
`createInitialAgentState(model: string): AgentState`, seeding all six fields (the five above
plus `citations`). Both hooks call it — `Main.tsx:12` and `ResearchCanvas.tsx:24` — so the two
seeds cannot drift by construction, and the return type makes a missing field a compile error
rather than a mount-order surprise.

A factory rather than a constant because `model` is only known at render time from
`useModelSelectorContext`; the comment above the function records that reason and the
same-shape requirement, so the next reader doesn't inline it back. Verified against current
source this pass. Note the seed now includes `citations`, which the original finding predates
— the field set tracks
[agent-state-shape-contract](/brain/rules/agent-state-shape-contract.md), so a new state key
has to be added here too.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/lib/types.ts:32](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/lib/types.ts#L32) — `createInitialAgentState`
[2] [src/app/Main.tsx:12](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/Main.tsx#L12) — `createInitialAgentState`
[3] [src/components/ResearchCanvas.tsx:24](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/components/ResearchCanvas.tsx#L24) — `createInitialAgentState`

<!-- okf:citations:end -->
