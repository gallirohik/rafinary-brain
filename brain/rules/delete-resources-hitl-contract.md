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
  - src/components/ResearchCanvas.tsx:38 :: DeleteResources
  - agents/python/src/lib/chat.py:32 :: DeleteResources
  - agents/python/src/lib/chat.py:87 :: DeleteResources
  - agents/python/src/lib/chat.py:156 :: DeleteResources
  - agents/typescript/src/agent.ts:55 :: DeleteResources
  - agents/typescript/src/chat.ts:42 :: DeleteResources
  - agents/typescript/src/chat.ts:43 :: DeleteResources
  - agents/typescript/src/chat.ts:62 :: DeleteResources
  - agents/typescript/src/chat.ts:95 :: DeleteResources
description: "The agent's DeleteResources tool routes to an interrupt, and the frontend renders the confirm dialog via useCopilotAction under the SAME string name — a mismatch means the delete never prompts and silently stalls"
tags: [agent-bridge]
timestamp: 2026-07-26T22:44:42.840Z
---
`DeleteResources` is the one tool in this app that renders **frontend** UI (a confirm
dialog) instead of executing on the backend — a human-in-the-loop pattern. The string name
is the coupling, and it must be identical on both sides.

Backend side (agent):
- The tool is declared as a no-op `@tool` and bound to the model (`chat.py:32,87`; TS
  `chat.ts:42-43,95`). It has no body — it exists only to be *called*, not run.
- When the model calls it, the chat node routes `goto="delete_node"` (`chat.py:156`; TS
  `agent.ts:55`), and the graph is compiled with `interrupt_after=["delete_node"]`
  (`agent.py` / `agent.ts` compile). The graph **pauses** there.
- The TS agent additionally declares `emitToolCalls: "DeleteResources"` in the chat node's
  CopilotKit config (`chat.ts:62`) so the pending call is streamed to the UI.

Frontend side:
- `useCopilotAction({ name: "DeleteResources", available: "remote", renderAndWait: ... })`
  (`ResearchCanvas.tsx:37-38`) catches that interrupt and renders the "Delete these resources?"
  dialog with Cancel/Delete buttons. `handler("YES"|"NO")` resumes the graph.
- On resume, `perform_delete_node` reads the tool message (`YES`/`NO`) and filters
  `resources` (`delete.py` / `delete.ts`).

If the frontend action name and the agent tool name drift apart, the graph interrupts but no
dialog ever appears — the run hangs with no error. This is the canonical example for
[adding a new cross-boundary tool](/brain/playbooks/add-agent-tool-howto.md).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/components/ResearchCanvas.tsx:38](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/components/ResearchCanvas.tsx#L38) — `DeleteResources`
[2] [agents/python/src/lib/chat.py:32](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L32) — `DeleteResources`
[3] [agents/python/src/lib/chat.py:87](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L87) — `DeleteResources`
[4] [agents/python/src/lib/chat.py:156](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L156) — `DeleteResources`
[5] [agents/typescript/src/agent.ts:55](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/agent.ts#L55) — `DeleteResources`
[6] [agents/typescript/src/chat.ts:42](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/chat.ts#L42) — `DeleteResources`
[7] [agents/typescript/src/chat.ts:43](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/chat.ts#L43) — `DeleteResources`
[8] [agents/typescript/src/chat.ts:62](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/chat.ts#L62) — `DeleteResources`
[9] [agents/typescript/src/chat.ts:95](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/chat.ts#L95) — `DeleteResources`

<!-- okf:citations:end -->
