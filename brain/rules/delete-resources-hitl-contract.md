---
schemaVersion: 1
id: delete-resources-hitl-contract
type: contract
domain: agent-bridge
title: "DeleteResources" is the human-in-the-loop tool name shared by agent and frontend
summary: The agent's DeleteResources tool routes to an interrupt, and the frontend renders the confirm dialog via useCopilotAction under the SAME string name — a mismatch means the delete never prompts and silently stalls
links: [research-chat-flow, agent-name-contract, add-agent-tool-howto]
anchor: DeleteResources
failure: silent
cites:
  - src/components/ResearchCanvas.tsx:39 :: DeleteResources
  - agents/python/src/lib/chat.py:32 :: DeleteResources
  - agents/python/src/lib/chat.py:82 :: DeleteResources
  - agents/python/src/lib/chat.py:150 :: DeleteResources
  - agents/typescript/src/agent.ts:55 :: DeleteResources
  - agents/typescript/src/chat.ts:37 :: DeleteResources
  - agents/typescript/src/chat.ts:38 :: DeleteResources
  - agents/typescript/src/chat.ts:57 :: DeleteResources
  - agents/typescript/src/chat.ts:82 :: DeleteResources
---
`DeleteResources` is the one tool in this app that renders **frontend** UI (a confirm
dialog) instead of executing on the backend — a human-in-the-loop pattern. The string name
is the coupling, and it must be identical on both sides.

Backend side (agent):
- The tool is declared as a no-op `@tool` and bound to the model (`chat.py:32,82`; TS
  `chat.ts:37-38,82`). It has no body — it exists only to be *called*, not run.
- When the model calls it, the chat node routes `goto="delete_node"` (`chat.py:150`; TS
  `agent.ts:55`), and the graph is compiled with `interrupt_after=["delete_node"]`
  (`agent.py` / `agent.ts` compile). The graph **pauses** there.
- The TS agent additionally declares `emitToolCalls: "DeleteResources"` in the chat node's
  CopilotKit config (`chat.ts:57`) so the pending call is streamed to the UI.

Frontend side:
- `useCopilotAction({ name: "DeleteResources", available: "remote", renderAndWait: ... })`
  (`ResearchCanvas.tsx:39`) catches that interrupt and renders the "Delete these resources?"
  dialog with Cancel/Delete buttons. `handler("YES"|"NO")` resumes the graph.
- On resume, `perform_delete_node` reads the tool message (`YES`/`NO`) and filters
  `resources` (`delete.py` / `delete.ts`).

If the frontend action name and the agent tool name drift apart, the graph interrupts but no
dialog ever appears — the run hangs with no error. This is the canonical example for
[adding a new cross-boundary tool](/brain/playbooks/add-agent-tool-howto.md).
