---
schemaVersion: 1
id: fact-check-node-unguarded-toolcall
priority: P2
category: correctness
status: open
title: fact_check_node indexes tool_calls[0] unguarded at four sites — the only node that breaks the repo's own guard convention
summary: search_node and perform_delete_node both check `if ai_message.tool_calls` before indexing, but fact_check_node — the newest node — reads ai_message.tool_calls[0]["id"] at four sites with no guard, so any path that reaches it without a live tool call raises IndexError and aborts the graph run
fix: Hoist one guard at the top of fact_check_node (bail early if `not ai_message.tool_calls`) and reuse the resolved id, mirroring search_node's shape (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [agent-python]
cites:
  - agents/python/src/lib/fact_check.py:65 :: tool_calls[0]
  - agents/python/src/lib/fact_check.py:114 :: tool_calls[0]
  - agents/python/src/lib/fact_check.py:129 :: tool_calls[0]
  - agents/python/src/lib/fact_check.py:141 :: tool_calls[0]
  - agents/python/src/lib/search.py:67 :: if not ai_message.tool_calls
  - agents/python/src/lib/delete.py:26 :: if ai_message.tool_calls
found: 2026-07-27
type: Improvement
description: "search_node and perform_delete_node both check `if ai_message.tool_calls` before indexing, but fact_check_node — the newest node — reads ai_message.tool_calls[0]['id'] at four sites with no guard, so any path that reaches it without a live tool call raises IndexError and aborts the graph run"
timestamp: 2026-07-27
---
The repo has an explicit defensive convention — *never index `tool_calls[0]` unguarded*,
because forced `tool_choice` is not honored identically by the four providers the app offers
([langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md),
[model-selection-flow](/brain/playbooks/model-selection-flow.md)). Three of the four consumer
nodes follow it; `fact_check_node` — the newest, added by the `verifiable-report` epic — does
not:

| node | incoming read | guarded? |
| --- | --- | --- |
| `search_node` | `search.py:74` | ✓ `search.py:67` |
| `search_node` (forced response) | `search.py:152` | ✓ `search.py:139` + resolving `ToolMessage` |
| `perform_delete_node` | `delete.py:27` | ✓ `delete.py:26`, with a `function_call` fallback |
| `fact_check_node` | `fact_check.py:65`, `:114`, `:129`, `:141` | ✗ all four unguarded |

All four unguarded sites read `ai_message.tool_calls[0]["id"]` off
`state["messages"][-1]` (`fact_check.py:54`) to stamp the resolving `ToolMessage`.

## Honest reachability — why P2, not P1

`fact_check_node` is entered only from `chat_node`'s router, which reaches it exclusively via
`ai_message.tool_calls[0]["name"] == "FactCheckReport"` and appends that same AIMessage to
state — so on the current graph the last message is structurally guaranteed to carry a tool
call, and **this is not a live crash today**. It is graded as convention drift plus a latent
footgun, not as a live break; the closed `search-node-unguarded-toolcall` row was P1 precisely
because its second site sat on a *forced-tool_choice response*, where a provider declining the
tool is an ordinary occurrence. Note `fact_check_node` already guards **that** class correctly
at `fact_check.py:126`.

What makes it worth a row anyway: the guarantee is a property of `chat.py`'s router, not of
this node, and nothing encodes it. Adding a second inbound edge to `fact_check_node`, resuming
from a checkpoint whose tail message differs, or copying this node as the template for the next
one (the brain warns to *check the code, not the convention*, because of exactly this file)
each turn a silent assumption into an `IndexError` that aborts the whole run. The fix is one
guard and a local variable, and it makes the node self-contained.

Mirror consideration: the TypeScript port has not yet ported `fact_check_node` at all
([agent-typescript-parity](/brain/rules/agent-typescript-parity.md)) — fixing Python first
means the port inherits the guarded shape rather than the drifted one.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/fact_check.py:65](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L65) — `tool_calls[0]`
[2] [agents/python/src/lib/fact_check.py:114](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L114) — `tool_calls[0]`
[3] [agents/python/src/lib/fact_check.py:129](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L129) — `tool_calls[0]`
[4] [agents/python/src/lib/fact_check.py:141](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L141) — `tool_calls[0]`
[5] [agents/python/src/lib/search.py:67](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L67) — `if not ai_message.tool_calls`
[6] [agents/python/src/lib/delete.py:26](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/delete.py#L26) — `if ai_message.tool_calls`

<!-- okf:citations:end -->
