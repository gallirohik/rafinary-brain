---
schemaVersion: 1
id: verifiable-report-factcheck-node
plan: verifiable-report
parent: verifiable-report
kind: task
title: Implement the fact-check tool + graph node (Python agent)
description: >-
  The actual "check the draft against the sources" behavior — a chat-triggered
  tool that routes to a dedicated node, checks each claim in `report` against
  the fetched resource content, and writes structured citations back to state.
approach: >-
  Follow the add-agent-tool-howto + search_node structured-extraction pattern:
  a no-op FactCheckReport @tool bound into chat_node's tool list
  (chat.py:77-84 bind_tools list); on call, chat_node's routing/goto logic
  (chat.py:116-154, widening the Command[Literal[...]] return type) dispatches
  to a new fact_check_node in a new agents/python/src/lib/fact_check.py; the
  node loads full resource content (reusing download.py's cache/lookup), calls
  get_model(state) with tool_choice forced onto a pydantic FactCheckResult
  schema (a list of ClaimCheck{claim, resource_urls, supported, note}), writes
  the result to state["citations"], streams progress via state["logs"] +
  copilotkit_emit_state (mirroring search_node), and — like search_node
  (search.py:143-148,156-161) — appends a resolving ToolMessage for the
  FactCheckReport tool_call_id before returning, since an unresolved tool_call
  causes a 400 "tool_calls must be followed by tool result messages" on the
  next chat_node invoke. add_node + the fact_check_node -> chat_node edge are
  wired in agent.py.
status: done
priority: 2
---

## Log

- **2026-07-24** — Implemented `agents/python/src/lib/fact_check.py` (new):
  `ClaimCheckInput` + internal `ExtractClaimChecks` forced-tool-choice tool
  (mirrors `search.py`'s `ExtractResources`), `fact_check_node` resolving the
  original `FactCheckReport` tool_call_id on every exit path. Wired
  `FactCheckReport` (outward no-op tool) + routing in `chat.py`, node + edge in
  `agent.py`. Commit `3112f4f` `[verifiable-report-factcheck-node] feat: add
  FactCheckReport node`. prism traced all three exit paths (empty report / no
  tool_calls back / happy path) and confirmed the ToolMessage resolution is
  airtight against the "tool_calls must be followed by tool result messages"
  400. Two non-blocking observations from review (not Done-check failures,
  left as-is): `fact_check_node` doesn't clear `state["logs"]` between runs
  like `search_node` does (cosmetic accumulation), and it relies on the
  routing invariant rather than asserting `tool_calls` is non-empty at the top
  (safe today, less defensive than `search_node`). Live run / `npm run build`
  not possible in this standalone checkout (no `node_modules`, `workspace:*`
  deps, no parent monorepo) — verified via `python3 -m py_compile` + full
  manual trace instead. Done-check: prism PASS.

## Done-check

- `agents/python/src/lib/fact_check.py` exists with a `FactCheckReport` no-op
  `@tool` and a `fact_check_node` following `search_node`'s structured
  extraction shape (forced `tool_choice`, pydantic arg schema).
- `chat.py` binds `FactCheckReport` in `chat_node`'s `bind_tools([...])` list
  (chat.py:77-84) and its goto/routing logic dispatches to `fact_check_node`
  on that tool call. `agent.py` adds the `fact_check_node` node + the
  `fact_check_node` → `chat_node` edge.
- `fact_check_node` appends a resolving `ToolMessage` for the
  `FactCheckReport` tool_call_id before returning (mirroring
  `search_node`, search.py:143-148,156-161) — verify by sending a second chat
  message right after a fact-check run and confirming it does NOT 400 with
  "tool_calls must be followed by tool result messages".
- Manual run (`npm run dev`): add ≥1 resource with known content, write a
  report containing one claim clearly supported by that resource and one that
  is not, ask the chat to fact-check the report — `state.citations` populates
  with both claims, the unsupported one flagged `supported: false`, and
  `state.logs` shows fact-check progress steps.
- `chat.py`'s existing "don't recite resources verbatim" instruction and
  `WriteReport`'s behavior are unchanged — this task only adds a new tool/node.
