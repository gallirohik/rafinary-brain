---
schemaVersion: 1
id: agent-typescript-parity
type: convention
domain: agent-typescript
title: The TypeScript agent is an alternate port of the Python agent (not run by default)
summary: agents/typescript mirrors an OLDER Python graph with @langchain/langgraph — 5 nodes, no fact_check_node, no citations state; it is a lightly-used alternate that npm run dev does NOT start, so treat Python as the source of truth and expect the port to lag
links: [langgraph-agent-convention, agent-state-shape-contract, build-tooling-convention, state-persistence-convention]
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
  call (`chat.ts:94`) passes a list that stops at `DeleteResources` (`chat.ts:95`).
- **No `citations` in state.** `AgentStateAnnotation` (`state.ts:19-26`) declares only
  `model`, `research_question`, `report`, `resources`, `logs` — see
  [the state shape contract](/brain/rules/agent-state-shape-contract.md).
- If you wire the TS agent back in as the live backend, the frontend's Fact Check panel
  simply never renders (the `state.citations` guard stays false) — silently, no error.
- **No checkpointer.** `agent.ts:8` imports `MemorySaver` and never uses it; the graph
  compiles with only `interruptAfter`. Python attaches one
  (`agent.py:48`), so the delete interrupt is resumable there and **not** here — see
  [state-persistence-convention](/brain/rules/state-persistence-convention.md).

**Which of those gaps the machine watches, and which it does not.** This note's whole value
is a list of things *absent* from `agents/typescript/`, and a stale absence claim is the most
dangerous kind — so read the four negative claims the bullets above make with this scoping in
mind. It is an explicit judgment, not an oversight:

| gap | gated? | how / why not |
| --- | --- | --- |
| no `FactCheckReport` tool in `chat.ts` | **yes** | [langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md) declares `anchor: FactCheckReport` and cites all three code occurrences (`chat.py:37`, `:88`, `:160`). A fourth occurrence anywhere — porting it to TS being the obvious one — fails the completeness gate every run |
| no `fact_check_node` (5 nodes vs 6) | no | `fact_check_node` occurs 6× in the **Python** agent |
| no `citations` in `AgentStateAnnotation` | no | `citations` occurs in `state.py`, `fact_check.py`, `types.ts`, `FactCheck.tsx`, `ResearchCanvas.tsx` |
| no active checkpointer | no | `MemorySaver` occurs 3× in `agent.py` **and once right here** — `agent.ts:8` imports it and never uses it, so even a token-presence check would read as "present" |

The reason the last three cannot be mechanized is structural, not effort: `absent:` is
**repo-wide**, and every one of these tokens legitimately exists elsewhere in the repo.
Declaring `absent: fact_check_node` would fail instantly and *falsely* — it would assert
something this brain knows to be untrue. The anchor substitute does not reach them either:
their occurrence sets live mostly in the Python agent and are not exhaustively cited here,
so an anchor would fail for the wrong reason. What is actually needed is a path-scoped form
(`absent: fact_check_node in agents/typescript/`), which the contract does not have.

**Consequence for you, stated plainly:** those three are **authored** claims — hand-verified
against `agents/typescript/src/{agent,chat,state}.ts` at each refresh (last: 2026-08-01), not
re-grepped by any gate. The `FactCheckReport` row is `verified`. If you are about to make a
decision on one of the three ungated rows, re-grep it under `agents/typescript/` yourself; it
is a two-second check and this note cannot do it for you.

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
port (`search.ts:53`, `search.ts:157`) — that fix was mirrored; fact-check was not.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/typescript/src/agent.ts:15](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/agent.ts#L15) — `StateGraph`
[2] [agents/typescript/src/agent.ts:23](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/agent.ts#L23) — `addConditionalEdges`
[3] [agents/typescript/src/agent.ts:34](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/agent.ts#L34) — `interruptAfter`
[4] [agents/typescript/src/model.ts:20](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/model.ts#L20) — `openai`
[5] [agents/typescript/src/model.ts:40](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/model.ts#L40) — `throw new Error`
[6] [agents/typescript/src/state.ts:19](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/state.ts#L19) — `AgentStateAnnotation`

<!-- okf:citations:end -->
