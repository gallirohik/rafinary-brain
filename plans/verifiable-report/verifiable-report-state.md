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
status: done
priority: 2
---

## Log

- **2026-07-24** — Implemented: `Citation` TypedDict (`agents/python/src/lib/state.py`)
  + matching `Citation` interface (`src/lib/types.ts`), `citations` field added to
  both `AgentState` shapes. TS agent (`agents/typescript/src/state.ts`)
  intentionally untouched per the logged "Python-only for now" decision.
  Commit `35dcc30` `[verifiable-report-state] feat: add Citation type to AgentState`.
  Build could not be run directly (`npm run build`) — this checkout is standalone
  (no `node_modules`, `@copilotkit/*` deps use `workspace:*`, `npm install` fails
  `EUNSUPPORTEDPROTOCOL`). prism independently confirmed the constraint (reproduced
  the same npm error, no lockfile present) and instead verified via real
  `@copilotkit/react-core` type defs (`initialState` types as `any`, `setState`
  call sites all use `...state` spreads) that the widened type cannot break
  either `useCoAgent<AgentState>` call site. Done-check: prism PASS.

## Done-check

- `agents/python/src/lib/state.py` declares a `Citation` TypedDict (`claim: str`,
  `resource_urls: List[str]`, `supported: bool`, `note: Optional[str]`) and
  `AgentState.citations: List[Citation]`.
- `src/lib/types.ts` declares a matching `Citation` interface and adds
  `citations: Citation[]` to the frontend `AgentState` type.
- `npm run build` passes with no type errors touching these fields.
- `Main.tsx` / `ResearchCanvas.tsx`'s `useCoAgent<AgentState>` call sites still
  compile against the widened type.
