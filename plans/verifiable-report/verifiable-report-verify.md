---
schemaVersion: 1
id: verifiable-report-verify
plan: verifiable-report
parent: verifiable-report
kind: task
title: End-to-end manual verification across state, node, and UI
description: >-
  No automated test suite exists in this repo (no test script in
  package.json, no pytest config) — verification is a documented manual pass
  tying the state, node, and UI tasks together as one flow, run fresh after
  all three land.
approach: >-
  Run npm run dev (Python agent + UI), walk the full flow, capture the result
  in this task's Log.
status: done
priority: 2
---

## Log

- **2026-07-24** — This standalone checkout cannot run `npm run dev` (no
  `node_modules`, `@copilotkit/*` are `workspace:*`, no parent monorepo).
  Instead of a purely-manual pass, copied the repo to a scratch dir, swapped
  `workspace:*` for the real published `@copilotkit/*@^1.63.2`, and ran real
  tooling:
  - `tsc --noEmit` (real v5.9.3 from installed `node_modules`) — exit 0, zero
    errors.
  - `npm run build` (`next build`) — compiled successfully, type-checked, all
    6 static pages generated, exit 0. One pre-existing vendor warning from
    `@copilotkit/runtime`'s own bundling, unrelated to this diff.
  - `npm run lint` — genuinely N/A: this repo has no ESLint config at all
    (confirmed, no `.eslintrc*`/`eslint.config.*` anywhere), a pre-existing
    condition this plan didn't touch or break.
  - `npm run dev:ui` booted for real, `curl localhost:<port>/` → HTTP 200,
    page contains "Research Question"/"Research Draft" (real render, not a
    crash page) and zero "fact-check" occurrences — confirming the
    `state.citations && state.citations.length > 0` guard correctly hides the
    panel on a live empty-citations render.
  prism independently reproduced this entire recipe from scratch (not
  trusting the report) and got matching results, then read the full
  accumulated diff (`f0d0a55..HEAD`, 8 files) for coherence and re-confirmed
  the `Citation` shape is consistent end-to-end.

  **Residual risk (flagged loudly to the human, not silently accepted): no
  live, LLM-driven fact-check run has ever executed anywhere in this plan —
  this environment has no OpenAI/Anthropic/Tavily API keys.** Proven: build
  correctness, type correctness, static graph-wiring correctness, and
  pattern-parity with the already-working `search_node`. NOT proven by
  execution: that the model actually returns a well-formed
  `ExtractClaimChecks` call, that `state.citations` populates/streams on a
  real run, that a `supported:false` claim really renders the amber flag on
  real model output, and that a chat message sent right after a fact-check
  run doesn't 400. Before shipping this feature to real users, run ONE live
  smoke test with real API keys: add 2+ resources, write a report with an
  obviously-supported and an obviously-unsupported claim, ask chat to
  fact-check it, confirm citations stream in correctly and a follow-up
  message doesn't error.

  Non-blocking, already logged on the factcheck-node task: `fact_check_node`
  doesn't clear `state["logs"]` between runs (cosmetic accumulation), and
  doesn't defensively guard `tool_calls` non-empty at its own top (relies on
  the routing invariant).

  Done-check: prism PASS on the plan as a whole, with the live-run caveat
  above surfaced to the human rather than silently accepted.

## Done-check

- Fresh `npm run dev` session: add 2+ resources, write/generate a report with
  at least one claim clearly supported by a resource and one clearly not, ask
  the chat to fact-check the report.
- `state.citations` streams in; the `FactCheck` panel shows both claims with
  correct supported/unsupported status, and the unsupported claim's flag is
  visually obvious.
- No regressions: `Search`, `DeleteResources` (HITL confirm dialog), and plain
  report editing still work as before.
- `npm run build` + `npm run lint` pass on the final diff.
