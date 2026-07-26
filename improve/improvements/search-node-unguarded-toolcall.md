---
schemaVersion: 1
id: search-node-unguarded-toolcall
priority: P1
category: correctness
status: fixed
title: search_node assumes the model always returns a tool call — an empty response kills the run
summary: search.py indexes tool_calls[0] with no guard; if a provider ignores the forced tool_choice (or returns text/refusal), an unhandled IndexError aborts the primary research flow mid-conversation
fix: Guard tool_calls before indexing in search_node — if empty, emit a graceful log/message and return instead of throwing (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [agent-python, agent-bridge]
cites:
  - agents/python/src/lib/search.py:74 :: tool_calls[0]
  - agents/python/src/lib/search.py:152 :: tool_calls[0]
  - agents/python/src/lib/search.py:145 :: tool_call_id
  - agents/typescript/src/search.ts:53 :: tool_calls?.length
found: 2026-07-20
fixed: 2026-07-23
fixed_by: search-guard
type: Improvement
description: "search.py indexes tool_calls[0] with no guard; if a provider ignores the forced tool_choice (or returns text/refusal), an unhandled IndexError aborts the primary research flow mid-conversation"
timestamp: 2026-07-20
---
`search_node` (the actively-run Python backend, see
[langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)) reads the incoming
tool call (`ai_message.tool_calls[0]["args"]["queries"]`, now `search.py:74`) and the model's
extraction response (`ai_message_response.tool_calls[0]["args"]["resources"]`, now
`search.py:152`) — originally with **no guard** that a tool call exists. The extraction call
uses `tool_choice="ExtractResources"` (`search.py:117`), but forced tool choice is not honored
identically across the four providers this app offers (openai / anthropic / google_genai / grok
— see [model-selection-flow](/brain/playbooks/model-selection-flow.md)); a safety refusal, an
empty-result completion, or a provider that returns text instead throws `IndexError` and the
whole graph run aborts with no user-facing result.

Compiles and runs fine on the happy path, so no gate catches it — this is exactly the silent
correctness gap the search flow ([research-chat-flow](/brain/playbooks/research-chat-flow.md))
depends on. `chat_node` already handles the no-tool-call case (falls through to `goto="__end__"`,
`chat.py:146`); `search_node` should be similarly defensive. Mirror the fix in the TS port
([agent-typescript-parity](/brain/rules/agent-typescript-parity.md)).

## Resolution (2026-07-23, plan `search-guard`)

Both indexing sites are now guarded in both ports; an empty/missing `tool_calls` response no
longer crashes the node — it appends a user-facing log entry (the app's established
`state["logs"]` convention, matching `chat_node`) and returns early.

- **Python** `agents/python/src/lib/search.py`:
  - Guard 1 (`search_node`, the incoming Search tool call, `:67-72`): `if not
    ai_message.tool_calls:` → log "No search queries provided; skipping search.", emit state,
    return. No dangling tool_call to resolve here (the incoming AIMessage carries none).
  - Guard 2 (the forced `ExtractResources` extraction response, `:139-150`): `if not
    ai_message_response.tool_calls:` → log "No resources extracted from search results.",
    **append a `ToolMessage` resolving the original Search `tool_call_id`** (`:145`), emit,
    return.
- **TypeScript** `agents/typescript/src/search.ts`: same two guards (`:53`, `:157`),
  returning partial state (`messages: [...]`) per the TS port's return-not-mutate convention.
  This is one of the few places the TS port is NOT behind Python
  ([agent parity](/brain/rules/agent-typescript-parity.md)).

The dangling-tool_call resolution in Guard 2 is load-bearing: the first Python attempt returned
without appending that `ToolMessage`, leaving the original Search `tool_call` unresolved — the
next `chat_node` model turn would 400 ("tool_calls must be followed by tool result messages").
**prism caught it**, it was fixed and re-validated PASS; the TS port applied the resolving
`ToolMessage` from the start and also passed prism validation.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/search.py:74](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L74) — `tool_calls[0]`
[2] [agents/python/src/lib/search.py:152](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L152) — `tool_calls[0]`
[3] [agents/python/src/lib/search.py:145](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L145) — `tool_call_id`
[4] [agents/typescript/src/search.ts:53](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/src/search.ts#L53) — `tool_calls?.length`

<!-- okf:citations:end -->
