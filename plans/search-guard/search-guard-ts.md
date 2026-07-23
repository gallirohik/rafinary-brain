---
schemaVersion: 1
id: search-guard-ts
plan: search-guard
parent: search-guard
kind: task
title: Guard the two tool_calls![0] index sites in the TypeScript search_node (parity)
description: >-
  Mirror the Python guard in the TS port. Replace the runtime-unsafe non-null
  assertions at search.ts:49 and search.ts:129 with real guards so an empty/missing
  tool_calls logs a warning and returns early instead of throwing at runtime.
approach: check `aiMessage.tool_calls?.length` before each index; push to logs and early-return the state object
status: done
priority: 2
---

## Context

- `agents/typescript/src/search.ts:49` — `const queries = aiMessage.tool_calls![0]["args"]["queries"];`
- `agents/typescript/src/search.ts:129` — `const newResources = aiMessageResponse.tool_calls![0]["args"]["resources"];`
- The `!` non-null assertion is compile-safe but runtime-unsafe — an empty array
  throws `TypeError` at runtime. In scope per rule `agent-typescript-parity` (TS
  mirrors Python node-for-node; lightly-used alternate).
- Match the node's existing return shape: search_node returns
  `{ messages, resources, logs }` objects (`search.ts:58-62,108-112,133-145`), and
  logs are `logs.push({ message, done })` (`search.ts:51-56,70`). Guarded early
  returns must return a valid state object of that shape.

## Approach

1. Before `search.ts:49`: if `!aiMessage.tool_calls?.length`, push a warning to
   `logs`, emit state via `copilotkitEmitState`, and return an early state object
   (`{ messages: [], resources, logs }`) — no queries to search.
2. Before `search.ts:129`: if `!aiMessageResponse.tool_calls?.length`, push a warning,
   and return the state object without the `resources.push(...)` and without the
   trailing ToolMessage that references `aiMessageResponse.tool_calls![0]["id"]`
   (`search.ts:138`).
3. Keep parity with the Python guard's behavior (log + early return, no retry).

## Done-check

- An `aiMessage` (or `aiMessageResponse`) with empty/undefined `tool_calls` no longer
  throws a runtime `TypeError`; instead a warning is pushed to `logs` and the node
  returns a valid `{ messages, resources, logs }` state object.
- Both index sites (`:49` and `:129`) are guarded; the runtime-unsafe `!` assertions
  at those two sites are removed (or made unreachable behind the guard).
- The existing normal-path flow (non-empty tool_calls) is unchanged.
- Behavior matches the Python child (`search-guard-py`) node-for-node per
  `agent-typescript-parity`.

## Log

- 2026-07-23 (atlas): Guarded both `tool_calls![0]` index sites in
  `agents/typescript/src/search.ts`, mirroring the Python child node-for-node
  (`agent-typescript-parity`).
  - Guard 1 (before the queries index, was `:49`): added
    `if (!aiMessage.tool_calls?.length)` — pushes
    `{ message: "No search queries provided; skipping search.", done: true }` to `logs`,
    emits state via `copilotkitEmitState` (destructuring `messages` out to match the
    node's emit shape), and returns early `{ messages: [], resources, logs }`. No
    tool_call exists in this branch, so nothing dangles — no ToolMessage needed (matches
    Python guard 1).
  - Guard 2 (before the resources index, was `:129`): added
    `if (!aiMessageResponse.tool_calls?.length)` — pushes
    `{ message: "No resources extracted from search results.", done: true }`, emits state,
    and returns early. CRITICAL LESSON from the sibling `search-guard-py` prism FAIL: its
    first pass skipped the trailing ToolMessage that resolves the ORIGINAL Search
    tool_call, leaving a dangling AIMessage tool_call that 400s the next model turn. TS
    has the same exposure — `agent.ts` wires `search_node -> download -> chat_node`
    unconditionally, and the happy-path `searchResultsToolMessage` (search.ts ~:84) is
    what normally resolves `aiMessage.tool_calls[0].id`. So guard 2's early return appends
    a `ToolMessage({ tool_call_id: aiMessage.tool_calls![0]["id"]!, name: "Search" })`
    (safe: guard 1 already proved `aiMessage.tool_calls` non-empty) so the message history
    stays valid for the next `chat_node` re-invoke.
  - TS-specific adaptation vs Python: the TS node returns a PARTIAL state whose `messages`
    is appended by the MessagesState reducer (via `CopilotKitStateAnnotation`), not a
    mutated-in-place whole state like Python — so guard returns carry only the NEW messages
    to append (guard 1: none; guard 2: the single resolving ToolMessage), not the full
    history.
  - Happy path untouched; the `!` assertions at both sites remain but are now unreachable
    when `tool_calls` is empty (behind the guards). Could not run `tsc` — the TS agent
    workspace has no installed deps (lightly-used alternate port); verified by review.
    Left `status: todo` for prism to gate.
- 2026-07-23 (prism): PASSED. Both guards correct; additive messages reducer confirmed
  (`messages: []` does not wipe state); guard 2 correctly resolves the original Search
  tool_call id via a ToolMessage before returning; node-for-node parity with the Python
  fix confirmed; no regressions. `status: done`.
