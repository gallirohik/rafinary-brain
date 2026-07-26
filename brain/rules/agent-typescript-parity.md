---
schemaVersion: 1
id: agent-typescript-parity
type: convention
domain: agent-typescript
title: The TypeScript agent is an alternate port of the Python agent (not run by default)
summary: agents/typescript mirrors an OLDER Python graph with @langchain/langgraph — 5 nodes, no fact_check_node, no citations state; it is a lightly-used alternate that npm run dev does NOT start, so treat Python as the source of truth and expect the port to lag
links: [langgraph-agent-convention, agent-state-shape-contract, build-tooling-convention]
cites:
  - agents/typescript/src/agent.ts:15 :: StateGraph
  - agents/typescript/src/agent.ts:23 :: addConditionalEdges
  - agents/typescript/src/agent.ts:34 :: interruptAfter
  - agents/typescript/src/model.ts:20 :: openai
  - agents/typescript/src/model.ts:40 :: throw new Error
  - agents/typescript/src/state.ts:19 :: AgentStateAnnotation
description: "agents/typescript mirrors an OLDER Python graph with @langchain/langgraph — 5 nodes, no fact_check_node, no citations state; it is a lightly-used alternate that npm run dev does NOT start, so treat Python as the source of truth and expect the port to lag"
tags: [agent-typescript]
timestamp: 2026-07-26T22:44:42.840Z
---
`agents/typescript/` is a port of the Python agent, not a separate feature — but it is a port
of an **older** revision of it. Use it only if you specifically run the JS backend, and read
the drift list below first.

**Which one actually runs:** `npm run dev` → `dev:agent` → `dev:agent:py`
(`uv run main.py`, `package.json:11-12`) — the **Python** agent on port 8000. The TS agent
has no `dev` wiring in the root `package.json`; it is installed separately
(`install:agent:ts` = `pnpm i`, `package.json:14`) and is reached only by pointing
`REMOTE_ACTION_URL` at a TS server you start yourself — the runtime route has one
`baseUrl` for both ports, not a commented-out JS branch
([route convention](/brain/rules/copilotkit-runtime-route-convention.md)). Treat Python as
the source of truth; the TS port lags.

**The port is currently BEHIND — known gaps** (all a logged `verifiable-report` plan
decision, "Python-only for now", not accidental rot):
- **No `fact_check_node`.** Python's graph has six nodes; the TS graph has five
  (`agent.ts:15-34`). There is no `FactCheckReport` tool in `chat.ts` — its `bindTools`
  list stops at `DeleteResources` (`chat.ts:82`).
- **No `citations` in state.** `AgentStateAnnotation` (`state.ts:19-26`) declares only
  `model`, `research_question`, `report`, `resources`, `logs` — see
  [the state shape contract](/brain/rules/agent-state-shape-contract.md).
- If you wire the TS agent back in as the live backend, the frontend's Fact Check panel
  simply never renders (the `state.citations` guard stays false) — silently, no error.

**Structural differences worth knowing** (the port is not line-identical):
- Graph edges use `addConditionalEdges("chat_node", route, [...])` with a standalone `route`
  function (`agent.ts:23`) instead of Python's `Command(goto=...)` returned from the node.
  The `route` fn dispatches on `message.constructor.name` (`"AIMessageChunk"` /
  `"ToolMessage"`) — a runtime-class check, more brittle than Python's explicit returns.
- State is a LangGraph `Annotation.Root` spreading `CopilotKitStateAnnotation.spec`
  (`state.ts:19-26`), vs Python's `MessagesState` TypedDict.
- `getModel` (`model.ts:20-40`) maps the same four providers but uses gpt-**4o** for openai
  (Python uses gpt-4o-**mini**) — a real drift to be aware of.

Everything in [the LangGraph agent convention](/brain/rules/langgraph-agent-convention.md)
(no-op tool schemas, get_model routing, SSRF-guarded download, Tavily search) applies here
too, in TS form. The empty-`tool_calls` guards added to `search_node` DO exist in the TS
port (`search.ts:53`, `search.ts:148-150`) — that fix was mirrored; fact-check was not.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/typescript/src/agent.ts:15](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/agent.ts#L15) — `StateGraph`
[2] [agents/typescript/src/agent.ts:23](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/agent.ts#L23) — `addConditionalEdges`
[3] [agents/typescript/src/agent.ts:34](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/agent.ts#L34) — `interruptAfter`
[4] [agents/typescript/src/model.ts:20](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/model.ts#L20) — `openai`
[5] [agents/typescript/src/model.ts:40](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/model.ts#L40) — `throw new Error`
[6] [agents/typescript/src/state.ts:19](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/state.ts#L19) — `AgentStateAnnotation`

<!-- okf:citations:end -->
