---
schemaVersion: 1
id: search-guard
plan: search-guard
parent: null
kind: epic
title: Guard search_node against missing tool_calls (Python + TS parity)
description: >-
  search_node in both agents indexes tool_calls[0] with no guard. The extraction
  call forces tool_choice="ExtractResources", but forced tool choice is not honored
  identically across the four providers this app offers (openai/anthropic/google_genai/
  grok) — a refusal, an empty completion, or a text-only response makes tool_calls
  empty and the whole graph run aborts (IndexError in Python, TypeError via the
  non-null assertion in TS) with no user-facing result. chat_node already handles the
  no-tool-call case defensively (falls through to goto="__end__"); search_node should
  be similarly defensive. Ledger: search-node-unguarded-toolcall (P1, correctness).
approach: guard both index sites in each agent; log via state logs + return unchanged instead of crashing
status: done
priority: 2
domains: [agent-python, agent-typescript]
---

## Context

Two unguarded index sites per agent:

- **Python** (`agents/python/src/lib/search.py`) — `search.py:66`
  (`ai_message.tool_calls[0]["args"]["queries"]`, the incoming Search call) and
  `search.py:130` (`ai_message_response.tool_calls[0]["args"]["resources"]`, the
  forced-extraction response at `search.py:109`). The second site is the real live
  risk; the first is defense-in-depth against a routing/caller change
  (`chat.py:147-148` only routes to search_node when a Search tool_call exists).
- **TypeScript** (`agents/typescript/src/search.ts`) — `search.ts:49` and
  `search.ts:129`, using the runtime-unsafe non-null assertion `tool_calls![0]`.
  In scope per rule `agent-typescript-parity` (the TS port mirrors Python
  node-for-node; lightly-used alternate, not run by `npm run dev`).

Both agents are the same 5-node StateGraph (rule `langgraph-agent-convention`).

## Alternatives considered

- **Re-prompt / retry on empty tool_calls** — rejected: heavier, changes flow
  behavior, out of scope for a P1 correctness guard. A graceful early return matches
  chat_node's existing idiom.
- **Raise a typed exception** — rejected: aborts the run just as loudly; the goal is
  a user-visible-safe degradation, not a different crash.

## Non-goals

- No change to the happy-path search/extraction flow.
- No change to AgentState shape, tool schemas, or agent-name wiring (no contract surface).
- No new retry/backoff machinery.

## Done-check (rollup)

Both child tasks pass their Done-checks; an empty/missing tool_calls at any of the
four index sites degrades gracefully (logged + early return) instead of aborting the
graph run, and the normal-path flow is byte-for-byte unchanged.

## Log

- 2026-07-23 (atlas, close-out): Both children prism-validated PASS; epic flipped to
  `status: done`.
  - `search-guard-py` needed one correction round: prism's first pass FAILED because
    guard 2's early return skipped the trailing `ToolMessage` that resolves the original
    Search `tool_call`, leaving a dangling AIMessage tool_call that would 400 the next
    `chat_node` re-invoke. Fixed by appending the resolving `ToolMessage` before the early
    return; re-validated PASS.
  - `search-guard-ts` applied that lesson from the start — its guard 2 resolved the
    original Search tool_call id via a `ToolMessage` in the first cut — and passed prism
    first try. Node-for-node parity with the Python fix confirmed (`agent-typescript-parity`).
  - All four index sites (py `:66`/`:130`, ts `:49`/`:129`) now degrade gracefully;
    happy path unchanged. Active plan pointer cleared.
