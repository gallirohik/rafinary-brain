---
schemaVersion: 1
id: submit-message-settimeout-race
priority: P3
lens: correctness
status: open
title: Chat submit clears logs then waits a hardcoded 30ms to sequence the state update
summary: onSubmitMessage clears coagent logs then awaits an arbitrary 30ms setTimeout to let the state flush before the run starts — a timing hack that can race on a slow render and leave stale logs visible for the first frames
fix: Sequence off the actual state commit rather than a fixed delay, or document why 30ms is required (~10 min)
leverage: { impact: low, effort: low }
blast_radius: [agent-bridge, components]
cites:
  - src/app/Main.tsx:55 :: setState
  - src/app/Main.tsx:56 :: setTimeout
found: 2026-07-20
---
On chat submit (`Main.tsx:53-57`) the handler clears the coagent `logs`
(`setState({ ...state, logs: [] })`, `Main.tsx:55`) then `await`s a hardcoded
`setTimeout(resolve, 30)` (`Main.tsx:56`) to give the state update time to propagate before the
run kicks off ([research-chat-flow](/brain/playbooks/research-chat-flow.md) step 1). The magic
30ms is a guess at React's commit latency — on a slow render it can fire before the clear lands,
briefly showing the previous run's Progress steps; on a fast one it is wasted latency. It is
silent (works most of the time), so nothing flags it. Prefer sequencing off the committed state
(effect/callback) over a fixed sleep, or at minimum comment why the delay exists so it is not
"cleaned up" into a regression.
