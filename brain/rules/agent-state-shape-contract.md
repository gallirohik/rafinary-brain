---
schemaVersion: 1
id: agent-state-shape-contract
type: contract
domain: state
title: AgentState is a cross-process shape shared by the frontend and both agents
summary: The five fields (model, research_question, report, resources, logs) are defined three times — TS frontend type, Python TypedDict, TS Annotation — and must stay in sync or streamed state silently fails to bind to the UI
links: [agent-name-contract, research-chat-flow, copilotkit-runtime-route-convention]
anchor: none  # cross-process shape — the FIELD SET is the contract, not one greppable token
failure: silent
cites:
  - src/lib/types.ts:7 :: AgentState
  - src/lib/types.ts:9 :: research_question
  - src/lib/types.ts:10 :: report
  - src/lib/types.ts:11 :: resources
  - src/lib/types.ts:12 :: logs
  - agents/python/src/lib/state.py:29 :: AgentState
  - agents/python/src/lib/state.py:36 :: research_question
  - agents/python/src/lib/state.py:39 :: logs
  - agents/typescript/src/state.ts:19 :: AgentStateAnnotation
  - agents/typescript/src/state.ts:21 :: research_question
  - agents/typescript/src/state.ts:24 :: logs
---
The coagent's shared state is the wire format between the LangGraph backend and the React
frontend. CopilotKit streams the graph's state object straight into the `useCoAgent` hook's
`state`, so a field the backend emits but the frontend doesn't expect (or vice versa) simply
doesn't render — no type error crosses the process boundary.

The five load-bearing fields — `model`, `research_question`, `report`, `resources`, `logs` —
are declared in **three** places that must agree:
- Frontend: `AgentState` type, `src/lib/types.ts:7-13` (note `resources`/`logs` are typed
  `any[]` here — the frontend is deliberately loose).
- Python agent: `AgentState(MessagesState)` TypedDict, `state.py:29-39`.
- TS agent: `AgentStateAnnotation` (LangGraph `Annotation.Root`), `state.ts:19-24`, which also
  spreads `CopilotKitStateAnnotation.spec` (the CopilotKit-managed message/emit plumbing).

Where each field is read/written on the frontend: `research_question` and `report` are
two-way-bound to inputs in `ResearchCanvas.tsx` (`:145`, `:195`); `resources` renders the
resource cards; `logs` drives the [Progress](/brain/rules/design-system-convention.md)
step list via `useCoAgentStateRender`. The backend writes them through tool-call
intermediate-state emission — see [the research chat flow](/brain/playbooks/research-chat-flow.md).

Gotcha: `research_question` and `report` are also the `state_key` targets of the
`emit_intermediate_state` config in the chat node (`chat.py:47,52`). Those string keys must
match a field name here too — a typo there breaks live streaming of that field silently.
Renaming any field is a five-place (frontend type + both agents + both emit configs) change.
`anchor: none` because the contract is the field SET, not a single token.
