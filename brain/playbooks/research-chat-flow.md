---
schemaVersion: 1
id: research-chat-flow
type: flow
domain: agent-bridge
title: "End-to-end — chat message to canvas update via the LangGraph research agent"
summary: "Traces a user message from CopilotChat through the /api/copilotkit runtime into the LangGraph graph (download→chat→search/delete/fact-check), and how tool-call intermediate state (or a direct node return, for fact-check) streams back to render the canvas, progress list, and fact-check panel"
links: [agent-name-contract, agent-state-shape-contract, copilotkit-runtime-route-convention, langgraph-agent-convention, delete-resources-hitl-contract, model-selection-flow]
cites:
  - src/app/Main.tsx:45 :: CopilotChat
  - src/app/api/copilotkit/route.ts:141 :: handleRequest
  - agents/python/src/lib/chat.py:82 :: bind_tools
  - agents/python/src/lib/chat.py:48 :: copilotkit_customize_config
  - agents/python/src/lib/chat.py:160 :: FactCheckReport
  - agents/python/src/lib/fact_check.py:137 :: citations
  - agents/python/src/lib/search.py:81 :: copilotkit_emit_state
  - src/components/ResearchCanvas.tsx:27 :: useCoAgentStateRender
  - src/components/ResearchCanvas.tsx:193 :: report
  - src/components/ResearchCanvas.tsx:202 :: citations
---
The load-bearing round trip. Follow it when debugging "I typed in chat and the canvas didn't
update" or "the report/resources/citations aren't streaming."

1. **User types** in `<CopilotChat>` (`Main.tsx:45`), the right-hand pane. On submit it
   clears `logs` in coagent state, then the message posts to `runtimeUrl` (`/api/copilotkit`,
   optionally `?lgcDeploymentUrl=`). The bound agent name comes from
   [the agent-name contract](/brain/rules/agent-name-contract.md).

2. **Runtime route** `POST /api/copilotkit` (`route.ts`) checks the cosmetic `x-api-key`,
   builds a `CopilotRuntime` whose `research_agent[_google_genai]` entries proxy to the
   backend (`LangGraphHttpAgent` → `REMOTE_ACTION_URL`, default localhost:8000), and calls
   `handleRequest(req)` (`route.ts:141`). `EmptyAdapter` means the runtime holds no LLM.
   If `?lgcDeploymentUrl=` rode along, the URL is DNS-resolved and rejected unless every
   resolved address is public (`route.ts:59-84`) before the runtime is rebuilt for
   LangGraph Cloud — a 400 here, not a hang.

3. **Graph executes** (`agents/python/src/agent.py`), entry `download` → `chat_node`.
   `chat_node` (`chat.py`) loads resource contents, calls `get_model(state)`, and
   **skips any resource whose cached content is the sentinel
   `"ERROR"`** (`chat.py:72`; `fact_check_node` does the same at `fact_check.py:81`). This
   is the debug branch for *"my resource is on the canvas but the agent never uses it"*:
   a failed download caches the literal string `"ERROR"` (`download.py:66,81`), and
   `download_node`'s re-fetch test `if not get_resource(...)` (`download.py:97`) is truthy
   for it, so **the URL is never retried for the life of the process** and no error reaches
   state or the UI — the log still says "done". Restart the agent to clear it; the full
   gotcha is in
   [langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md).
   `bind_tools([Search, WriteReport, WriteResearchQuestion, DeleteResources,
   FactCheckReport])` (`chat.py:82`). The model responds with a tool call; the node routes:
   - `WriteReport` / `WriteResearchQuestion` → update that state field, loop back to
     `chat_node`.
   - `Search` → `search_node` (Tavily search + `ExtractResources`) → back to `download`
     (fetch the new URLs).
   - `DeleteResources` → `delete_node`, which **interrupts** for frontend confirmation
     (see [the HITL contract](/brain/rules/delete-resources-hitl-contract.md)).
   - `FactCheckReport` (`chat.py:160`) → `fact_check_node` — checks every claim in
     `state.report` against the fetched resource content via a forced-tool-choice
     `ExtractClaimChecks` call, writes `state["citations"]` (`fact_check.py:137`), resolves
     the original tool call with a `ToolMessage`, and loops back to `chat_node`. No
     interrupt — unlike delete, it needs no human confirmation.
   - no tool → END.

4. **State streams back.** Nodes emit partial state as it's produced:
   `copilotkit_customize_config(emit_intermediate_state=[...])` maps a tool argument to a
   state key (`chat.py:48` — `WriteReport.report` → `state.report`), and `search_node`/
   `download_node` push progress with `copilotkit_emit_state(config, state)` (`search.py:81`).
   `fact_check_node` is different: `citations` isn't a single streamed tool argument, so the
   node sets `state["citations"]` directly and returns the whole state (same shape as
   `search_node`'s resource-list return) rather than using an `emit_intermediate_state`
   mapping. The frontend `useCoAgent` `state` updates live either way.

5. **Canvas renders.** `ResearchCanvas` two-way-binds `state.research_question` and
   `state.report` to its inputs (`ResearchCanvas.tsx:193`), lists `state.resources` as cards,
   `useCoAgentStateRender` (`:27`) renders `state.logs` through the `Progress` step list
   (component roster in
   [the design-system convention](/brain/rules/design-system-convention.md)) during
   search/download/fact-check, and a `state.citations && state.citations.length > 0` guard (`:202`) renders
   the `FactCheck` claim list once `fact_check_node` has populated it.

If a field doesn't render, the break is almost always a name mismatch in step 4's emit
config, the [state shape](/brain/rules/agent-state-shape-contract.md), or the
[agent name](/brain/rules/agent-name-contract.md). For `citations` specifically, also check
that `fact_check_node` actually resolved the `FactCheckReport` tool_call_id with a
`ToolMessage` — an unresolved tool call 400s the NEXT chat turn instead of failing loudly here.
If instead the turn died with an `IndexError` inside `fact_check_node`, that is the known
unguarded `tool_calls[0]` read (`fact_check.py:65,114,129,141`) — the node does **not**
follow the guard convention the other three consumer nodes do; see
[langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md).
