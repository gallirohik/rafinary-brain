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
  - src/app/api/copilotkit/route.ts:109 :: research_agent
  - src/app/api/copilotkit/route.ts:110 :: research_agent
  - src/app/api/copilotkit/route.ts:112 :: research_agent
  - src/app/api/copilotkit/route.ts:113 :: research_agent
  - src/app/api/copilotkit/route.ts:121 :: research_agent
  - src/app/api/copilotkit/route.ts:124 :: research_agent
  - src/app/api/copilotkit/route.ts:126 :: research_agent
  - src/app/api/copilotkit/route.ts:129 :: research_agent
  - agents/python/langgraph.json:6 :: research_agent
  - agents/python/langgraph.json:7 :: research_agent
  - agents/python/main.py:20 :: research_agent
  - agents/python/main.py:22 :: research_agent
  - agents/python/main.py:27 :: research_agent
  - agents/python/main.py:29 :: research_agent
  - agents/typescript/langgraph.json:6 :: research_agent
  - agents/typescript/langgraph.json:7 :: research_agent
description: "'research_agent' / 'research_agent_google_genai' must match across the frontend agent prop, the runtime registry, and each agent's langgraph.json + endpoint path, or the coagent silently never connects"
tags: [agent-bridge]
timestamp: 2026-07-26T22:44:42.840Z
---
A single string identifies the coagent, and it must be **identical** across every site
below or `useCoAgent` silently binds to nothing — no error, the chat just never drives the
canvas.

The chain, front to back:
1. The frontend picks the name: `model-selector-provider.tsx`
   ([model selection flow](/brain/playbooks/model-selection-flow.md) §2)
   sets `agent = "research_agent"`, switching to `"research_agent_google_genai"` only when
   `model === "google_genai"` (`model-selector-provider.tsx:42,44`). That value flows to
   `<CopilotKit agent={agent}>` (`page.tsx:31`) and to `useCoAgent({ name: agent })` in
   `Main.tsx:11` / `ResearchCanvas.tsx:23,28` (the second is `useCoAgentStateRender`, which
   takes the same `name`).
2. The runtime registers agents under those exact keys (`route.ts:109,112` HTTP path mode,
   `route.ts:121,126` LangGraph-Cloud mode, with `graphId` at `:124,:129`). A key the
   frontend never asks for is dead; a name the frontend asks for that isn't registered 404s.
3. Each backend declares the graph under the same id in `langgraph.json`
   (`agents/python/langgraph.json:6-7`, `agents/typescript/langgraph.json:6-7`) — and the
   Python server additionally mounts the FastAPI **endpoint path** with the name baked in:
   `LangGraphAGUIAgent(name=...)` + `path="/copilotkit/agents/research_agent[...]"`
   (`main.py:20,22,27,29`). The runtime's `LangGraphHttpAgent` url (`route.ts:110,113`) must
   match that path exactly.

Non-obvious: **both** graph ids (`research_agent` and `research_agent_google_genai`) resolve
to the *same* compiled `graph` object in both backends — the second name is not a different
agent. Actual model choice is not driven by the agent name; it rides the agent **state**
(`state.model`) into `get_model`. See [model selection flow](/brain/playbooks/model-selection-flow.md).

To add or rename an agent you must touch all five files in lockstep. See
[the runtime route convention](/brain/rules/copilotkit-runtime-route-convention.md) and
[the research chat flow](/brain/playbooks/research-chat-flow.md).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/lib/model-selector-provider.tsx:42](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/lib/model-selector-provider.tsx#L42) — `research_agent`
[2] [src/lib/model-selector-provider.tsx:44](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/lib/model-selector-provider.tsx#L44) — `research_agent`
[3] [src/app/api/copilotkit/route.ts:109](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L109) — `research_agent`
[4] [src/app/api/copilotkit/route.ts:110](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L110) — `research_agent`
[5] [src/app/api/copilotkit/route.ts:112](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L112) — `research_agent`
[6] [src/app/api/copilotkit/route.ts:113](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L113) — `research_agent`
[7] [src/app/api/copilotkit/route.ts:121](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L121) — `research_agent`
[8] [src/app/api/copilotkit/route.ts:124](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L124) — `research_agent`
[9] [src/app/api/copilotkit/route.ts:126](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L126) — `research_agent`
[10] [src/app/api/copilotkit/route.ts:129](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L129) — `research_agent`
[11] [agents/python/langgraph.json:6](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/langgraph.json#L6) — `research_agent`
[12] [agents/python/langgraph.json:7](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/langgraph.json#L7) — `research_agent`
[13] [agents/python/main.py:20](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L20) — `research_agent`
[14] [agents/python/main.py:22](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L22) — `research_agent`
[15] [agents/python/main.py:27](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L27) — `research_agent`
[16] [agents/python/main.py:29](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L29) — `research_agent`
[17] [agents/typescript/langgraph.json:6](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/langgraph.json#L6) — `research_agent`
[18] [agents/typescript/langgraph.json:7](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/langgraph.json#L7) — `research_agent`

<!-- okf:citations:end -->
