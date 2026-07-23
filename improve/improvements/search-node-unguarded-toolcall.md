---
schemaVersion: 1
id: search-node-unguarded-toolcall
priority: P1
lens: correctness
status: open
title: search_node assumes the model always returns a tool call — an empty response kills the run
summary: search.py indexes tool_calls[0] with no guard; if a provider ignores the forced tool_choice (or returns text/refusal), an unhandled IndexError aborts the primary research flow mid-conversation
fix: Guard tool_calls before indexing in search_node — if empty, emit a graceful log/message and return instead of throwing (~10 min)
leverage: { impact: medium, effort: low }
blast_radius: [agent-python, agent-bridge]
cites:
  - agents/python/src/lib/search.py:66 :: tool_calls[0]
  - agents/python/src/lib/search.py:130 :: tool_calls[0]
found: 2026-07-20
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
