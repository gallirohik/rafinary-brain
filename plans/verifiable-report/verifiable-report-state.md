---
schemaVersion: 1
id: verifiable-report-state
plan: verifiable-report
parent: verifiable-report
kind: task
title: Add a Citation type + citations field to AgentState (Python agent + frontend)
description: >-
  The fact-check node needs somewhere to write its output, and the frontend
  needs a typed shape to render it. AgentState has no field today connecting a
  report claim to the resource(s) behind it.
approach: >-
  Define Citation{claim, resource_urls, supported, note} in
  agents/python/src/lib/state.py, add citations: List[Citation] to AgentState;
  mirror as a real Citation interface + citations: Citation[] in
  src/lib/types.ts (frontend AgentState). The TypeScript agent's state.ts is
  intentionally NOT touched — TS parity is deferred (see the epic's Decisions).
status: todo
priority: 2
---

## Done-check

- `agents/python/src/lib/state.py` declares a `Citation` TypedDict (`claim: str`,
  `resource_urls: List[str]`, `supported: bool`, `note: Optional[str]`) and
  `AgentState.citations: List[Citation]`.
- `src/lib/types.ts` declares a matching `Citation` interface and adds
  `citations: Citation[]` to the frontend `AgentState` type.
- `npm run build` passes with no type errors touching these fields.
- `Main.tsx` / `ResearchCanvas.tsx`'s `useCoAgent<AgentState>` call sites still
  compile against the widened type.
