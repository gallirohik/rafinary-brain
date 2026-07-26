---
schemaVersion: 1
id: agent-state-shape-contract
type: contract
domain: state
title: "AgentState is a cross-process shape shared by the frontend and both agents"
summary: "Six fields (model, research_question, report, resources, logs, citations) are defined across the TS frontend type and the Python agent; citations is Python+frontend ONLY (the TS agent's state.ts intentionally lags, verifiable-report plan decision) — a field the backend emits but the frontend doesn't expect (or vice versa) simply doesn't render"
links: [agent-name-contract, research-chat-flow, copilotkit-runtime-route-convention]
anchor: none  # cross-process shape — the FIELD SET is the contract, not one greppable token
cites:
  - src/lib/types.ts:19 :: AgentState
  - src/lib/types.ts:21 :: research_question
  - src/lib/types.ts:22 :: report
  - src/lib/types.ts:23 :: resources
  - src/lib/types.ts:24 :: logs
  - src/lib/types.ts:25 :: citations
  - agents/python/src/lib/state.py:41 :: AgentState
  - agents/python/src/lib/state.py:48 :: research_question
  - agents/python/src/lib/state.py:51 :: logs
  - agents/python/src/lib/state.py:52 :: citations
  - agents/typescript/src/state.ts:19 :: AgentStateAnnotation
  - agents/typescript/src/state.ts:21 :: research_question
  - agents/typescript/src/state.ts:24 :: logs
---
The coagent's shared state is the wire format between the LangGraph backend and the React
frontend. CopilotKit streams the graph's state object straight into the `useCoAgent` hook's
`state`, so a field the backend emits but the frontend doesn't expect (or vice versa) simply
doesn't render — no type error crosses the process boundary.

The six load-bearing fields — `model`, `research_question`, `report`, `resources`, `logs`,
`citations` — are declared in the Python agent and the frontend, both of which must agree:
- Frontend: `AgentState` type, `src/lib/types.ts:19-26` (`resources: Resource[]`,
  `logs: Log[]`, `citations: Citation[]` — all three are real types now, not `any[]`; a
  `createInitialAgentState(model)` factory, `types.ts:32-41`, is the single seed used by
  both `useCoAgent` call sites, `Main.tsx` and `ResearchCanvas.tsx`).
- Python agent: `AgentState(MessagesState)` TypedDict, `state.py:41-52`.

**`citations` is Python + frontend ONLY — the TS agent is NOT in sync, intentionally.**
`agents/typescript/src/state.ts`'s `AgentStateAnnotation` still declares only the original
five fields. This is a logged plan decision (`verifiable-report` epic, "Python-only for
now") rather than drift: the TS port is an accepted-lagging alternate per
[agent-typescript-parity](/brain/rules/agent-typescript-parity.md) and isn't run by default.
If the TS agent is ever wired back in, `citations: List[Citation]` must be added there too
before this contract goes back to "all three agree."

Where each field is read/written on the frontend: `research_question` and `report` are
two-way-bound to inputs in `ResearchCanvas.tsx`; `resources` renders the resource cards;
`logs` drives the [Progress](/brain/rules/design-system-convention.md) step list via
`useCoAgentStateRender`; `citations` drives `FactCheck.tsx`'s claim list (gated on
`state.citations && state.citations.length > 0` — a defensive guard, since `citations` is only
ever populated once `fact_check_node` runs, so an earlier/partial coagent-state snapshot can
still arrive without it, non-optional TS type notwithstanding). The backend writes these
fields through tool-call intermediate-state emission and direct state return — see
[the research chat flow](/brain/playbooks/research-chat-flow.md).

Gotcha: `research_question` and `report` are also the `state_key` targets of the
`emit_intermediate_state` config in the chat node. Those string keys must match a field name
here too — a typo there breaks live streaming of that field silently. `citations` is NOT an
`emit_intermediate_state` target — `fact_check_node` sets `state["citations"]` directly and
returns it, rather than streaming it progressively as a tool argument (there's no single
tool-argument shape to map; the whole claims list lands at once). Renaming any field is now
a multi-place change (frontend type + Python agent + emit configs for the streamed fields);
`anchor: none` because the contract is the field SET, not a single token.
