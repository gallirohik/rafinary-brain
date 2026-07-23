---
schemaVersion: 1
id: agent-name-contract
type: contract
domain: agent-bridge
title: The agent-name strings wire the frontend to a LangGraph graph across 5 files
summary: "research_agent" / "research_agent_google_genai" must match across the frontend agent prop, the runtime registry, and each agent's langgraph.json + endpoint path, or the coagent silently never connects
links: [agent-state-shape-contract, research-chat-flow, copilotkit-runtime-route-convention, model-selection-flow]
anchor: research_agent
failure: silent
cites:
  - src/lib/model-selector-provider.tsx:42 :: research_agent
  - src/lib/model-selector-provider.tsx:44 :: research_agent
  - src/app/api/copilotkit/route.ts:73 :: research_agent
  - src/app/api/copilotkit/route.ts:74 :: research_agent
  - src/app/api/copilotkit/route.ts:76 :: research_agent
  - src/app/api/copilotkit/route.ts:77 :: research_agent
  - src/app/api/copilotkit/route.ts:85 :: research_agent
  - src/app/api/copilotkit/route.ts:88 :: research_agent
  - src/app/api/copilotkit/route.ts:90 :: research_agent
  - src/app/api/copilotkit/route.ts:93 :: research_agent
  - agents/python/langgraph.json:6 :: research_agent
  - agents/python/langgraph.json:7 :: research_agent
  - agents/python/main.py:20 :: research_agent
  - agents/python/main.py:22 :: research_agent
  - agents/python/main.py:27 :: research_agent
  - agents/python/main.py:29 :: research_agent
  - agents/typescript/langgraph.json:6 :: research_agent
  - agents/typescript/langgraph.json:7 :: research_agent
---
A single string identifies the coagent, and it must be **identical** across every site
below or `useCoAgent` silently binds to nothing — no error, the chat just never drives the
canvas.

The chain, front to back:
1. The frontend picks the name: [model-selector-provider](/brain/rules/copilotkit-runtime-route-convention.md)
   sets `agent = "research_agent"`, switching to `"research_agent_google_genai"` only when
   `model === "google_genai"` (`model-selector-provider.tsx:42,44`). That value flows to
   `<CopilotKit agent={agent}>` (`page.tsx:31`) and to `useCoAgent({ name: agent })` in
   `Main.tsx:11` / `ResearchCanvas.tsx:22,29`.
2. The runtime registers agents under those exact keys (`route.ts:73,76` HTTP path mode,
   `route.ts:85,90` LangGraph-Cloud mode, with `graphId` at `:88,:93`). A key the frontend
   never asks for is dead; a name the frontend asks for that isn't registered 404s.
3. Each backend declares the graph under the same id in `langgraph.json`
   (`agents/python/langgraph.json:6-7`, `agents/typescript/langgraph.json:6-7`) — and the
   Python server additionally mounts the FastAPI **endpoint path** with the name baked in:
   `LangGraphAGUIAgent(name=...)` + `path="/copilotkit/agents/research_agent[...]"`
   (`main.py:20,22,27,29`). The runtime's `LangGraphHttpAgent` url (`route.ts:74,77`) must
   match that path exactly.

Non-obvious: **both** graph ids (`research_agent` and `research_agent_google_genai`) resolve
to the *same* compiled `graph` object in both backends — the second name is not a different
agent. Actual model choice is not driven by the agent name; it rides the agent **state**
(`state.model`) into `get_model`. See [model selection flow](/brain/playbooks/model-selection-flow.md).

To add or rename an agent you must touch all five files in lockstep. See
[the runtime route convention](/brain/rules/copilotkit-runtime-route-convention.md) and
[the research chat flow](/brain/playbooks/research-chat-flow.md).
