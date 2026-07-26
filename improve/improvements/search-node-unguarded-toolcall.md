---
schemaVersion: 1
id: search-node-unguarded-toolcall
priority: P1
lens: correctness
status: fixed
title: search_node assumes the model always returns a tool call — an empty response kills the run
summary: search.py indexes tool_calls[0] with no guard; if a provider ignores the forced tool_choice (or returns text/refusal), an unhandled IndexError aborts the primary research flow mid-conversation
fix: Guard tool_calls before indexing in search_node — if empty, emit a graceful log/message and return instead of throwing (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [agent-python, agent-bridge]
cites:
  - agents/python/src/lib/search.py:66 :: tool_calls[0]
  - agents/python/src/lib/search.py:130 :: tool_calls[0]
found: 2026-07-20
fixed: 2026-07-23
fixed_by: search-guard
---
`search_node` (the actively-run Python backend, see
[langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)) reads the incoming
tool call at `search.py:66` (`ai_message.tool_calls[0]["args"]["queries"]`) and the model's
extraction response at `search.py:130` (`ai_message_response.tool_calls[0]["args"]["resources"]`)
with **no guard** that a tool call exists. The extraction call uses
`tool_choice="ExtractResources"` (`search.py:109`), but forced tool choice is not honored
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
  - Guard 1 (`search_node`, the incoming Search tool call): `if not ai_message.tool_calls:` →
    log "No search queries provided; skipping search.", emit state, return. No dangling
    tool_call to resolve here (the incoming AIMessage carries none).
  - Guard 2 (the forced `ExtractResources` extraction response): `if not
    ai_message_response.tool_calls:` → log "No resources extracted from search results.",
    **append a `ToolMessage` resolving the original Search `tool_call_id`**, emit, return.
- **TypeScript** `agents/typescript/src/search.ts`: same two guards, returning partial state
  (`messages: [...]`) per the TS port's return-not-mutate convention.

The dangling-tool_call resolution in Guard 2 is load-bearing: the first Python attempt returned
without appending that `ToolMessage`, leaving the original Search `tool_call` unresolved — the
next `chat_node` model turn would 400 ("tool_calls must be followed by tool result messages").
**prism caught it**, it was fixed and re-validated PASS; the TS port applied the resolving
`ToolMessage` from the start and also passed prism validation.
