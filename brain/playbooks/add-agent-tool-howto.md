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
  - agents/python/src/lib/chat.py:77 :: bind_tools
  - agents/python/src/lib/chat.py:150 :: DeleteResources
  - agents/python/src/lib/chat.py:47 :: state_key
  - src/components/ResearchCanvas.tsx:38 :: useCopilotAction
---
Adding a tool means touching a few coordinated sites. Pick the pattern by what the tool does.

**Every tool (backend, `agents/python/src/lib/chat.py`):**
1. Declare it as an empty `@tool` schema (`chat.py:16-38`) — no body; args define the
   contract. (TS: `tool(() => {}, { name, schema })` in `chat.ts`.)
2. Add it to the `bind_tools([...])` list in the chat node (`chat.py:77`).
3. Handle its call: inspect `ai_message.tool_calls[0]["name"]` and either update state or
   route to a node (`chat.py:150` shows the `goto` dispatch). Add a graph node + edges in
   `agent.py` if it needs its own step.
4. Mirror all of the above in the **TypeScript** agent to keep parity (see
   [agent parity](/brain/rules/agent-typescript-parity.md)).

**If the tool STREAMS a state field to the canvas** (like `WriteReport`): add an
`emit_intermediate_state` entry mapping `{ state_key, tool, tool_argument }`
(`chat.py:47`). The `state_key` must be a real field in
[AgentState](/brain/rules/agent-state-shape-contract.md), defined in all three state files,
or nothing renders.

**If the tool DRIVES frontend UI / needs human confirmation** (like `DeleteResources`):
1. Compile the graph so it interrupts after the handling node (`interrupt_after=[...]`).
2. On the frontend, register `useCopilotAction({ name: "<ToolName>", available: "remote",
   renderAndWait })` in `ResearchCanvas.tsx` (`:38`) — the `name` **must** equal the tool
   name exactly. See [the DeleteResources HITL contract](/brain/rules/delete-resources-hitl-contract.md)
   for the full pattern; a mismatch hangs silently.

Verify end to end against [the research chat flow](/brain/playbooks/research-chat-flow.md):
send a message that triggers the tool and confirm state streams and (if applicable) the UI
prompts.
