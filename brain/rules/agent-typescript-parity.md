---
schemaVersion: 1
id: agent-typescript-parity
type: convention
domain: agent-typescript
title: The TypeScript agent is an alternate port of the Python agent (not run by default)
summary: agents/typescript mirrors the Python graph node-for-node with @langchain/langgraph; it is a lightly-used alternate — npm run dev runs the PYTHON agent, and the TS one is wired via pnpm + an explicit route.ts uncomment
links: [langgraph-agent-convention, agent-state-shape-contract, build-tooling-convention]
cites:
  - agents/typescript/src/agent.ts:15 :: StateGraph
  - agents/typescript/src/agent.ts:23 :: addConditionalEdges
  - agents/typescript/src/agent.ts:34 :: interruptAfter
  - agents/typescript/src/model.ts:20 :: openai
  - agents/typescript/src/model.ts:40 :: throw new Error
  - agents/typescript/src/state.ts:19 :: AgentStateAnnotation
---
`agents/typescript/` is a **faithful port** of the Python agent, not a separate feature. Same
five nodes, same tool names, same AgentState fields, same model options. Use it only if you
specifically run the JS backend.

**Which one actually runs:** `npm run dev` → `dev:agent` → `dev:agent:py`
(`uv run main.py`) — the **Python** agent on port 8000. The TS agent has no `dev` wiring in
the root `package.json`; it is installed separately (`install:agent:ts` = `pnpm i`) and, per
`readme.md`, requires uncommenting the JS remoteEndpoints path in
[route.ts](/brain/rules/copilotkit-runtime-route-convention.md). Treat Python as the source
of truth; the TS port can lag.

**Structural differences worth knowing** (the port is not line-identical):
- Graph edges use `addConditionalEdges("chat_node", route, [...])` with a standalone `route`
  function (`agent.ts:23`) instead of Python's `Command(goto=...)` returned from the node.
  The `route` fn dispatches on `message.constructor.name` (`"AIMessageChunk"` /
  `"ToolMessage"`) — a runtime-class check, more brittle than Python's explicit returns.
- State is a LangGraph `Annotation.Root` spreading `CopilotKitStateAnnotation.spec`
  (`state.ts:19-29`), vs Python's `MessagesState` TypedDict.
- `getModel` (`model.ts:20-40`) maps the same four providers but uses gpt-**4o** for openai
  (Python uses gpt-4o-**mini**) — a real drift to be aware of.

Everything in [the LangGraph agent convention](/brain/rules/langgraph-agent-convention.md)
(no-op tool schemas, get_model routing, SSRF-guarded download, Tavily search) applies here
too, in TS form.
