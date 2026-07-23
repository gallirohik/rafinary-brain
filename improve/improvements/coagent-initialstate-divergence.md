---
schemaVersion: 1
id: coagent-initialstate-divergence
priority: P2
lens: architecture
status: open
title: Two useCoAgent hooks for the same agent seed different initialState
summary: Main.tsx seeds the full AgentState (model, research_question, resources, report, logs) while ResearchCanvas.tsx seeds only { model } for the same agent name — a silent inconsistency where the initial shape depends on mount order
fix: Share one initialState (extract a constant or seed the full shape in both) so the two hooks agree (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [state, agent-bridge]
cites:
  - src/app/Main.tsx:12 :: initialState
  - src/components/ResearchCanvas.tsx:23 :: initialState
found: 2026-07-20
---
Both `Main` and `ResearchCanvas` call `useCoAgent<AgentState>({ name: agent })` against the
**same** coagent ([agent-name-contract](/brain/rules/agent-name-contract.md)), but pass
different `initialState`: `Main.tsx:12-18` seeds all five fields (`model`, `research_question`,
`resources`, `report`, `logs`), while `ResearchCanvas.tsx:23-25` seeds only `{ model }`. Whichever
hook initializes the shared state first wins, so the canvas's starting shape is order-dependent
rather than declared. It works today because `Main` mounts around `ResearchCanvas`, but this is a
latent footgun: reorder the tree and `resources`/`report`/`logs` start `undefined`, which the
render code papers over with `|| []` / `|| ""` guards. Define the initial state once and use it in
both places so the contract is explicit, not incidental.

