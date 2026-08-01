---
schemaVersion: 1
id: research-chat-flow
type: flow
domain: agent-bridge
title: "End-to-end — chat message to canvas update via the LangGraph research agent"
summary: "Authoritative trace (tight, cite-proven) of a CopilotChat turn through the /api/copilotkit runtime into the LangGraph agent graph and how streamed state signals render the canvas (download → chat → search/delete/fact-check → frontend render)."
links: [agent-name-contract, agent-state-shape-contract, copilotkit-runtime-route-convention, langgraph-agent-convention, delete-resources-hitl-contract, model-selection-flow]
cites:
  - src/app/Main.tsx:45 :: CopilotChat
  - src/app/api/copilotkit/route.ts:135 :: runtime construction / deploymentUrl
  - agents/python/src/lib/chat.py:82 :: resource aggregation
  - agents/python/src/lib/chat.py:160 :: tool-call routing / ToolMessage
  - agents/python/src/lib/search.py:81 :: copilotkit_emit_state / logs append
  - agents/python/src/lib/fact_check.py:137 :: state["citations"] assignment
  - src/components/ResearchCanvas.tsx:27 :: useCoAgentStateRender (Progress)
  - src/components/ResearchCanvas.tsx:193 :: report binding (Textarea value={state.report || ""})
  - src/components/ResearchCanvas.tsx:202 :: citations guard (FactCheck render)
description: "Authoritative, cite-driven trace of the observable round-trip: user message → runtime → LangGraph agent nodes → streamed intermediate state → canvas render. Each numbered step below cites the exact file:line proven at merge SHA 1ebb712b4a31969dd2ade1b59f92b18d1f456033. Do not rely on this playbook for implementation details beyond these exact cited lines; consult the files when changing behaviour. Note: one previously-cited anchor (agents/python/src/lib/chat.py:48) could not be verified at this SHA and was removed from the cites list; if that symbol is required, re-run verify-citations and re-anchor to the correct line."
tags: [agent-bridge]
timestamp: 2026-08-01T00:00:00.000Z
---

This playbook documents only the observable signals in the merged code and points you to the exact lines that prove each step.

1) User input and submit (frontend)
- What happens: The CopilotChat widget clears the co-agent logs immediately before starting a new turn.
- Evidence (quote): src/app/Main.tsx:45 shows the submit handler clearing logs:
  - "// clear the logs before starting the new research"
  - "setState({ ...state, logs: [] });" (src/app/Main.tsx:45)

2) Runtime route and runtime construction (server)
- What happens: The runtime route conditionally constructs a CopilotRuntime that wires a LangGraphAgent when a deploymentUrl is present — the runtime.agent entry includes the provided deploymentUrl.
- Evidence (quote): src/app/api/copilotkit/route.ts (context around line 135):
  - "if (deploymentUrl) {"
  - "  runtime = new CopilotRuntime({"
  - "    agents: {"
  - "      research_agent: new LangGraphAgent({"
  - "        deploymentUrl,"
  - "        langsmithApiKey,"
  - "        graphId: \"research_agent\","
  - "      }),"
  (src/app/api/copilotkit/route.ts:135)

3) Agent graph: resource aggregation and tool-call routing
- What happens: The chat node aggregates resource content into the run state (truncating by remaining budget) and the handler constructs ToolMessage responses and branches on tool_call names to route to search/delete/fact-check logic.
- Evidence (quote): agents/python/src/lib/chat.py:82 (resource aggregation loop):
  - "for resource in state[\"resources\"]:"
  - "    content = get_resource(resource[\"url\"])"
  - "    if content == \"ERROR\":"
  - "        continue"
  - "    if remaining <= 0:"
  - "        break"
  - "    content = content[:remaining]"
  - "    remaining -= len(content)"
  - "    resources.append({**resource, \"content\": content})"
  (agents/python/src/lib/chat.py:82)

- Evidence (quote): agents/python/src/lib/chat.py:160 (ToolMessage construction and tool call check):
  - "ToolMessage( ... tool_call_id=ai_message.tool_calls[0][\"id\"], content=\"Research question written.\" ),"
  - "if ai_message.tool_calls and ai_message.tool_calls[0][\"name\"] == \"Search\":"
  (agents/python/src/lib/chat.py:160)

4) Streaming intermediate state back to the frontend
- What happens: Nodes append progress entries into state["logs"] and call the runtime-state emitter so the frontend can render intermediate progress.
- Evidence (quote): agents/python/src/lib/search.py:81 shows logs appended and an emit call:
  - "for query in queries:"
  - "    state[\"logs\"].append({\"message\": f\"Search for {query}\", \"done\": False})"
  - "await copilotkit_emit_state(config, state)"
  (agents/python/src/lib/search.py:81)

- Fact-check handling (observable signal): the fact-check node writes claims into state["citations"] and appends a completion ToolMessage; that state assignment is the frontend-visible signal that citations are present.
- Evidence (quote): agents/python/src/lib/fact_check.py:137:
  - "claims = ai_message_response.tool_calls[0][\"args\"][\"claims\"]"
  - "state[\"citations\"] = claims"
  - "state[\"messages\"].append( ToolMessage( tool_call_id=ai_message.tool_calls[0][\"id\"], content=f\"Fact-check complete: {len(claims)} claim(s) checked.\", ), )"
  (agents/python/src/lib/fact_check.py:137)

5) Frontend rendering (canvas)
- What happens: ResearchCanvas binds to the co-agent state; Progress renders when state.logs exists; the report textarea binds to state.report; FactCheck renders only when state.citations is a non-empty array.
- Evidence (quote): src/components/ResearchCanvas.tsx:27 — render returns Progress when logs exist:
  - "if (!state.logs || state.logs.length === 0) {"
  - "  return null;"
  - "}"
  - "return <Progress logs={state.logs} />;"
  (src/components/ResearchCanvas.tsx:27)

- Evidence (quote): src/components/ResearchCanvas.tsx:193 — report textarea binds value={state.report || ""} and updates state on change:
  - "<Textarea ... value={state.report || \"\"} onChange={(e) => setState({ ...state, report: e.target.value })} ... />"
  (src/components/ResearchCanvas.tsx:193)

- Evidence (quote): src/components/ResearchCanvas.tsx:202 — FactCheck guarded render:
  - "{state.citations && state.citations.length > 0 && ( <div> ... <FactCheck citations={state.citations} /> ... )}"
  (src/components/ResearchCanvas.tsx:202)

Diagnosis checklist (how to debug common failures)
- If logs/progress don't appear: confirm nodes append to state["logs"] and call copilotkit_emit_state (agents/python/src/lib/search.py:81), and confirm the frontend's useCoAgentStateRender hook is active (src/components/ResearchCanvas.tsx:27).
- If report/resources don't update: confirm the chat node aggregates resources into state.resources (agents/python/src/lib/chat.py:82) and that the run writes the intended state keys (compare agent emit mappings and frontend state keys).
- If citations (FactCheck) don't appear: confirm the fact-check node wrote into state["citations"] (agents/python/src/lib/fact_check.py:137) and that the frontend guard at src/components/ResearchCanvas.tsx:202 will render when that array is non-empty.

What changed in this rewrite
- Removed a previously-cited anchor that could not be verified at the merge SHA (agents/python/src/lib/chat.py:48). The note now cites only anchors I verified and quotes the exact lines that prove the observable behavior. If you need the symbol previously referenced at chat.py:48 (copilotkit_customize_config), re-run verify-citations and provide the updated anchor so I can fold it back with an exact quote.

Graph context: this playbook links to several rules and playbooks (agent-name-contract, agent-state-shape-contract, copilotkit-runtime-route-convention, langgraph-agent-convention, delete-resources-hitl-contract, model-selection-flow). Those dependent notes should continue to reference this playbook by id; I did not change the id or domain. If you want the playbook to assert additional implementation details (specific guard functions or header semantics) include the exact route.ts line numbers and I will fold them in with direct quotes.