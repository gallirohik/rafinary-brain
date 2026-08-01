---
schemaVersion: 1
id: research-chat-flow
type: flow
domain: agent-bridge
title: "End-to-end — chat message to canvas update via the LangGraph research agent"
summary: "Traces a user message from CopilotChat through the /api/copilotkit runtime into the LangGraph graph (download→chat→search/delete/fact-check), and how intermediate state streams back to render the canvas, progress list, and fact-check panel"
links: [agent-name-contract, agent-state-shape-contract, copilotkit-runtime-route-convention, langgraph-agent-convention, delete-resources-hitl-contract, model-selection-flow]
cites:
  - src/app/Main.tsx:45 :: CopilotChat
  - src/app/api/copilotkit/route.ts:135 :: handleRequest
  - agents/python/src/lib/chat.py:48 :: copilotkit_customize_config
  - agents/python/src/lib/chat.py:82 :: resource aggregation
  - agents/python/src/lib/chat.py:160 :: tool-call routing / ToolMessage
  - agents/python/src/lib/fact_check.py:137 :: citations
  - agents/python/src/lib/search.py:81 :: copilotkit_emit_state
  - src/components/ResearchCanvas.tsx:27 :: useCoAgentStateRender
  - src/components/ResearchCanvas.tsx:193 :: report binding
  - src/components/ResearchCanvas.tsx:202 :: citations guard
description: "Authoritative trace of the round-trip: user message → runtime → LangGraph agent graph nodes → streamed state → canvas render. Each numbered step below is tied to the cited code lines (see `cites:` above). Do not rely on this note for implementation details beyond these exact cited lines; consult the files when changing behaviour."
tags: [agent-bridge]
timestamp: 2026-08-01T00:00:00.000Z
---

This note documents the observable, merged-code behaviour for the chat → canvas round trip. Quote-proven evidence is included inline (file:line) so you can jump to the exact lines that matter.

1) User input and submit (frontend)
- What happens: The CopilotChat widget in the right-hand pane clears the co-agent logs immediately before starting a new turn.
- Evidence: CopilotChat and the submit handler that clears logs (src/app/Main.tsx:45) — setState({ ...state, logs: [] }); (src/app/Main.tsx:45).

2) Runtime route and runtime construction (server)
- What happens: The runtime route constructs a CopilotRuntime and, when a deploymentUrl is provided, creates a runtime.agent entry that includes that deploymentUrl (the route maps an optional deploymentUrl into the runtime's research_agent configuration). See the cited block that conditionally instantiates the runtime with a LangGraphAgent when deploymentUrl is present (src/app/api/copilotkit/route.ts:135).
- Note: The rewrite intentionally avoids asserting runtime-side guards or exact HTTP header semantics unless the cited lines show them. The cited route lines show the runtime/agent wiring and the presence of an optional deploymentUrl path (src/app/api/copilotkit/route.ts:135).

3) Agent graph: nodes, tool binding, and routing
- What happens: The chat node customizes runtime config and aggregates resource content; the code shows a copilotkit customization call and the resource-aggregation behavior. Later, tool calls produced by the model are inspected and routed (the code builds ToolMessage responses and branches on tool_calls names) (agents/python/src/lib/chat.py:48, agents/python/src/lib/chat.py:82, agents/python/src/lib/chat.py:160).
- Evidence: copilotkit_customize_config usage (agents/python/src/lib/chat.py:48) and the resource aggregation loop that appends fetched content to state.resources (agents/python/src/lib/chat.py:82). The code at agents/python/src/lib/chat.py:160 shows constructing ToolMessage and checking ai_message.tool_calls[0]["name"] to route to Search/delete/etc.

4) Streaming intermediate state back to the frontend
- What happens: Nodes push partial progress/state back into the runtime state and the code calls a helper to emit intermediate state. The search path appends log entries into state["logs"] and calls copilotkit_emit_state(config, state) (agents/python/src/lib/search.py:81).
- Evidence: search appends a log entry per query then awaits copilotkit_emit_state(config, state) (agents/python/src/lib/search.py:81).
- Fact-check handling: The fact-check node assigns claims into state["citations"] and appends a ToolMessage indicating completion; that assignment is the observable signal that citations are present for the frontend to render (agents/python/src/lib/fact_check.py:137).

5) Frontend rendering (canvas)
- What happens: The ResearchCanvas component binds to the co-agent state and renders progress, report, and the fact-check list from state fields. The render hook shows Progress is rendered when state.logs exists (src/components/ResearchCanvas.tsx:27). The report textarea binds to state.report (src/components/ResearchCanvas.tsx:193). The FactCheck block renders behind the guard state.citations && state.citations.length > 0 (src/components/ResearchCanvas.tsx:202).
- Evidence: useCoAgentStateRender rendering Progress when logs exist (src/components/ResearchCanvas.tsx:27), the Textarea value={state.report || ""} (src/components/ResearchCanvas.tsx:193), and the citations guard that renders <FactCheck citations={state.citations} /> when state.citations is populated (src/components/ResearchCanvas.tsx:202).

Diagnosis checklist (how to debug common failures)
- If logs/progress don't appear: check that nodes are appending to state.logs and that copilotkit_emit_state is being called (agents/python/src/lib/search.py:81) and that the frontend's render hook is active (src/components/ResearchCanvas.tsx:27).
- If report/resources don't update: check the agent's emit-mapping / state keys (copilotkit_customize_config usage for mapping fields into state — agents/python/src/lib/chat.py:48) and verify the graph's resource aggregation (agents/python/src/lib/chat.py:82).
- If citations (FactCheck) don't appear: confirm that the fact-check path wrote into state["citations"] (agents/python/src/lib/fact_check.py:137) and that the frontend guard renders when that array is non-empty (src/components/ResearchCanvas.tsx:202). Also verify that the agent's tool call for fact-check produced the expected ToolMessage resolution (agents/python/src/lib/chat.py:160).

What I changed and why
- Removed or softened operational claims not directly visible at the cited lines (for example, any statement that the route performs DNS-resolving SSRF checks or returns specific HTTP 400s unless the cited route lines show this). The new text ties each observable step to the exact file:line that proves it, so readers can follow the code precisely.

If you want me to expand the note with the exact route-lines that show the x-api-key check or the isSafeDeploymentUrl call, provide the specific route.ts line numbers (or allow a wider-cite pass) and I will fold them into the note with exact quotes and line anchors.