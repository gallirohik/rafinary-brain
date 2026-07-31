---
schemaVersion: 1
id: search-node-unguarded-toolcall
priority: P1
category: correctness
status: fixed
title: search_node assumes the model always returns a tool call — an empty response kills the run
summary: search.py indexed tool_calls[0] with no guard; if a provider ignored the forced tool_choice (or returned text/refusal), an unhandled IndexError aborted the primary research flow mid-conversation — both indexing sites now carry an early-return guard
fix: Guard tool_calls before indexing in search_node — if empty, emit a graceful log/message and return instead of throwing (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [agent-python, agent-bridge]
cites:
  - agents/python/src/lib/search.py:67 :: if not ai_message.tool_calls
  - agents/python/src/lib/search.py:139 :: if not ai_message_response.tool_calls
found: 2026-07-20
fixed: 2026-07-23
fixed_by: search-guard
type: Improvement
description: "search.py indexed tool_calls[0] with no guard; if a provider ignored the forced tool_choice (or returned text/refusal), an unhandled IndexError aborted the primary research flow mid-conversation — both indexing sites now carry an early-return guard"
tags: [correctness, P1]
timestamp: 2026-07-20
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

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/search.py:67](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/search.py#L67) — `if not ai_message.tool_calls`
[2] [agents/python/src/lib/search.py:139](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/search.py#L139) — `if not ai_message_response.tool_calls`

<!-- okf:citations:end -->

Re-verified 2026-07-30: both guards are present in current source (`search.py:67`, `:139`).
This row's citations were re-pointed at the guards in the same pass — they previously named
the pre-fix indexing lines (`:66`, `:130`), which the edit shifted onto a blank line and a
bracket. `verify-citations` skips `status: fixed` rows, so that rot was invisible to the gate;
citing the fix rather than the removed defect keeps a tombstone checkable.
