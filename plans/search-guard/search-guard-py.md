---
schemaVersion: 1
id: search-guard-py
plan: search-guard
parent: search-guard
kind: task
title: Guard the two tool_calls[0] index sites in the Python search_node
description: >-
  Guard search.py:66 (incoming Search call) and search.py:130 (forced-extraction
  response) so an empty/missing tool_calls logs a warning and returns the state
  unchanged instead of raising IndexError and aborting the run.
approach: check `if not <msg>.tool_calls` before each index; append to state["logs"] and early-return state
status: done
priority: 2
---

## Context

- `agents/python/src/lib/search.py:66` — `queries = ai_message.tool_calls[0]["args"]["queries"]`
- `agents/python/src/lib/search.py:130` — `resources = ai_message_response.tool_calls[0]["args"]["resources"]`
  (response from the `tool_choice="ExtractResources"` bind at `search.py:109`)
- Mirror the defensive idiom already in `agents/python/src/lib/chat.py:146-148`
  (`goto="__end__"` when there is no matching tool_call) and the graceful-degradation
  style in `async_tavily_search` (`search.py:53-54`, catch → surface, don't crash).
- Logging mechanism: `state["logs"].append({"message": ..., "done": True})` (as used
  at `search.py:71,88`); `state["logs"]` and `state["resources"]` are already
  initialized at `search.py:64-65`.

## Approach

1. Before `search.py:66`: if `not ai_message.tool_calls`, append a warning log to
   `state["logs"]`, emit state, and `return state` (nothing to search — no queries).
2. Before `search.py:130`: if `not ai_message_response.tool_calls`, append a warning
   log, emit state, and `return state` (search ran but no resources extracted — skip
   the `resources.extend` and the trailing ToolMessage append at `search.py:132-139`).
3. Do not weaken or reorder any happy-path statement.

## Done-check

- An `ai_message` (or `ai_message_response`) with `tool_calls == []` or `None` no
  longer raises `IndexError`; instead a warning is appended to `state["logs"]` and the
  node returns the current state (graph run continues / ends cleanly, no abort).
- Both index sites (`:66` and `:130`) are guarded.
- The existing normal-path flow (non-empty tool_calls) is unchanged — queries are
  searched, resources extracted and extended, ToolMessages appended exactly as before.
- Manual/quick check: invoking search_node with a stubbed AIMessage whose `tool_calls`
  is empty returns without exception at each site.

## Log

- 2026-07-23 (atlas): Guarded both `tool_calls[0]` index sites in
  `agents/python/src/lib/search.py`.
  - Before the queries index (was `:66`): added `if not ai_message.tool_calls:` guard
    that appends `{"message": "No search queries provided; skipping search.", "done": True}`
    to `state["logs"]`, awaits `copilotkit_emit_state(config, state)`, and returns `state`
    early — before the `tool_calls[0]` deref and before the query-log fan-out.
  - Before the resources index (was `:130`): added `if not ai_message_response.tool_calls:`
    guard that appends `{"message": "No resources extracted from search results.", "done": True}`,
    awaits `copilotkit_emit_state(config, state)`, and returns `state` early — skipping the
    `resources.extend` and trailing ToolMessage append.
  - Happy path untouched; log-dict shape matches the existing `state["logs"].append(...)`
    idiom in this file. Left `status: todo` for prism to gate.
- 2026-07-23 (atlas, correction): prism FAILED the first pass — guard 2's early return
  skipped the trailing `ToolMessage` that resolves the original Search `tool_call` in
  `state["messages"]`, leaving a dangling AIMessage tool_call that would 400 the next
  `chat_node` re-invoke ("tool_calls must be followed by tool result messages"). Fixed:
  guard 2 now appends `ToolMessage(tool_call_id=ai_message.tool_calls[0]["id"],
  content="No resources extracted from search results.")` before the early `return state`
  (safe: guard 1 already proved `ai_message.tool_calls` non-empty). Guard 1 unchanged —
  in that case `tool_calls` is empty, so there is no tool_call to resolve.
- 2026-07-23 (prism, PASS): Re-validated and PASSED. Both `tool_calls[0]` index sites
  guarded; guard 2 resolves the original Search tool_call via `ToolMessage` before the
  early return; happy path unchanged; guard 1 correctly stays a no-op. No regressions.
  Status flipped to `done`.
