---
schemaVersion: 1
id: add-agent-tool-howto
type: how-to
domain: agent-bridge
title: How to add a new agent tool (and wire it to the canvas)
summary: Add the no-op @tool schema, bind it in the chat node, route on its name, and — if it drives UI or streams a state field — mirror the string on the frontend (useCopilotAction) or in an emit_intermediate_state mapping
links: [langgraph-agent-convention, delete-resources-hitl-contract, agent-state-shape-contract, research-chat-flow]
cites:
  - agents/python/src/lib/chat.py:16 :: @tool
  - agents/python/src/lib/chat.py:82 :: bind_tools
  - agents/python/src/lib/chat.py:156 :: DeleteResources
  - agents/python/src/lib/chat.py:52 :: state_key
  - src/components/ResearchCanvas.tsx:37 :: useCopilotAction
description: "Add the no-op @tool schema, bind it in the chat node, route on its name, and — if it drives UI or streams a state field — mirror the string on the frontend (useCopilotAction) or in an emit_intermediate_state mapping"
tags: [agent-bridge]
timestamp: 2026-07-26T22:44:42.840Z
---
Adding a tool means touching a few coordinated sites. Pick the pattern by what the tool does.

**Every tool (backend, `agents/python/src/lib/chat.py`):**
1. Declare it as an empty `@tool` schema (`chat.py:16-38`) — no body; args define the
   contract. (TS: `tool(() => {}, { name, schema })` in `chat.ts`.)
2. Add it to the `bind_tools([...])` list in the chat node (`chat.py:82-89`).
3. Handle its call: inspect `ai_message.tool_calls[0]["name"]` and either update state or
   route to a node (`chat.py:152-162` shows the `goto` dispatch chain; `:156` is the
   `DeleteResources` branch). If it needs its own step, add the node in `agent.py`
   (`add_node`) **and — the step that is easy to miss — add the node name to
   `chat_node`'s return annotation at `chat.py:43`**,
   `Command[Literal[..., "__end__"]]`. That `Literal` is the *only* declaration of
   `chat_node`'s legal destinations; `agent.py` carries no `add_conditional_edges` at all.
   Omit it and the `goto` silently fails to route — no mypy, no tests, no CI will catch it
   (see [langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)). Static
   edges into/out of your node still go in `agent.py` via `add_edge`.
   **Always guard `tool_calls` before indexing `[0]`** — a provider can return no
   tool call even under forced `tool_choice`; `search_node` learned this the hard way and
   now guards both sites, and any node that resolves a caller's `tool_call_id` must still
   append the matching `ToolMessage` on the early-return path or the NEXT chat turn 400s.
   **Copy `search_node` (`search.py:67,139`), NOT `fact_check_node`** — the newest node is
   the one that breaks this rule: it indexes the incoming `ai_message.tool_calls[0]["id"]`
   unguarded at `fact_check.py:65`, `:114`, `:129` and `:141` (its only guard, `:126`,
   covers the forced *response*, not the incoming call). The node-by-node context is in
   [langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md).
4. Decide explicitly about the **TypeScript** agent. Python is the live backend; the TS port
   is an accepted-lagging alternate that already lacks `FactCheckReport` and `citations`
   ([agent parity](/brain/rules/agent-typescript-parity.md)). Mirror the tool there if you
   want parity — but if you skip it, say so in the plan, the way the `verifiable-report`
   epic did, so the gap is a decision and not silent rot.

**If the tool STREAMS a state field to the canvas** (like `WriteReport`): add an
`emit_intermediate_state` entry mapping `{ state_key, tool, tool_argument }`
(`chat.py:48-62`; `state_key` at `:52` and `:57`). The `state_key` must be a real field in
[AgentState](/brain/rules/agent-state-shape-contract.md), defined in all three state files,
or nothing renders.

**If the tool DRIVES frontend UI / needs human confirmation** (like `DeleteResources`):
1. Compile the graph so it interrupts after the handling node (`interrupt_after=[...]`).
2. On the frontend, register `useCopilotAction({ name: "<ToolName>", available: "remote",
   renderAndWait })` in `ResearchCanvas.tsx` (`:37-83`) — the `name` (`:38`) **must** equal the tool
   name exactly. See [the DeleteResources HITL contract](/brain/rules/delete-resources-hitl-contract.md)
   for the full pattern; a mismatch hangs silently.

Verify end to end against [the research chat flow](/brain/playbooks/research-chat-flow.md):
send a message that triggers the tool and confirm state streams and (if applicable) the UI
prompts.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/chat.py:16](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L16) — `@tool`
[2] [agents/python/src/lib/chat.py:82](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L82) — `bind_tools`
[3] [agents/python/src/lib/chat.py:156](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L156) — `DeleteResources`
[4] [agents/python/src/lib/chat.py:52](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L52) — `state_key`
[5] [src/components/ResearchCanvas.tsx:37](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/components/ResearchCanvas.tsx#L37) — `useCopilotAction`

<!-- okf:citations:end -->
