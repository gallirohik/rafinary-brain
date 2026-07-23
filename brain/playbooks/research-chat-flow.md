---
schemaVersion: 1
id: research-chat-flow
type: flow
domain: agent-bridge
title: End-to-end — chat message to canvas update via the LangGraph research agent
summary: Traces a user message from CopilotChat through the /api/copilotkit runtime into the LangGraph graph (download→chat→search/delete), and how tool-call intermediate state streams back to render the canvas and progress list
links: [agent-name-contract, agent-state-shape-contract, copilotkit-runtime-route-convention, langgraph-agent-convention, delete-resources-hitl-contract, model-selection-flow]
cites:
  - src/app/Main.tsx:51 :: CopilotChat
  - src/app/api/copilotkit/route.ts:105 :: handleRequest
  - agents/python/src/lib/chat.py:77 :: bind_tools
  - agents/python/src/lib/chat.py:43 :: copilotkit_customize_config
  - agents/python/src/lib/search.py:73 :: copilotkit_emit_state
  - src/components/ResearchCanvas.tsx:28 :: useCoAgentStateRender
  - src/components/ResearchCanvas.tsx:195 :: report
---
The load-bearing round trip. Follow it when debugging "I typed in chat and the canvas didn't
update" or "the report/resources aren't streaming."

1. **User types** in `<CopilotChat>` (`Main.tsx:51`), the right-hand pane. On submit it
   clears `logs` in coagent state, then the message posts to `runtimeUrl` (`/api/copilotkit`,
   optionally `?lgcDeploymentUrl=`). The bound agent name comes from
   [the agent-name contract](/brain/rules/agent-name-contract.md).

2. **Runtime route** `POST /api/copilotkit` (`route.ts`) checks the cosmetic `x-api-key`,
   builds a `CopilotRuntime` whose `research_agent[_google_genai]` entries proxy to the
   backend (`LangGraphHttpAgent` → `REMOTE_ACTION_URL`, default localhost:8000), and calls
   `handleRequest(req)` (`route.ts:105`). `EmptyAdapter` means the runtime holds no LLM.

3. **Graph executes** (`agents/python/src/agent.py`), entry `download` → `chat_node`.
   `chat_node` (`chat.py`) loads resource contents, calls `get_model(state)`, and
   `bind_tools([Search, WriteReport, WriteResearchQuestion, DeleteResources])` (`chat.py:77`).
   The model responds with a tool call; the node routes:
   - `WriteReport` / `WriteResearchQuestion` → update that state field, loop back to
     `chat_node`.
   - `Search` → `search_node` (Tavily search + `ExtractResources`) → back to `download`
     (fetch the new URLs).
   - `DeleteResources` → `delete_node`, which **interrupts** for frontend confirmation
     (see [the HITL contract](/brain/rules/delete-resources-hitl-contract.md)).
   - no tool → END.

4. **State streams back.** Nodes emit partial state as it's produced:
   `copilotkit_customize_config(emit_intermediate_state=[...])` maps a tool argument to a
   state key (`chat.py:43` — `WriteReport.report` → `state.report`), and `search_node`/
   `download_node` push progress with `copilotkit_emit_state(config, state)` (`search.py:73`).
   The frontend `useCoAgent` `state` updates live.

5. **Canvas renders.** `ResearchCanvas` two-way-binds `state.research_question` and
   `state.report` to its inputs (`ResearchCanvas.tsx:195`), lists `state.resources` as cards,
   and `useCoAgentStateRender` (`:28`) renders `state.logs` through the
   [Progress](/brain/rules/design-system-convention.md) step list during search/download.

If a field doesn't render, the break is almost always a name mismatch in step 4's emit
config, the [state shape](/brain/rules/agent-state-shape-contract.md), or the
[agent name](/brain/rules/agent-name-contract.md).
