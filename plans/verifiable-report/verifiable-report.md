---
schemaVersion: 1
id: verifiable-report
plan: verifiable-report
parent: null
kind: epic
title: Verifiable report — citation-linked claims + a fact-check graph node
description: >-
  The report is free text with zero structural link to the resources backing
  it — nothing stops a claim from being written with no source behind it. This
  adds a fact-check graph node that checks the draft against the fetched
  resource content, plus a citations layer the frontend renders so every claim
  shows what backs it (or that nothing does).
priority: 2
status: done
domains: [state, agent-python, agent-bridge, components]
branch: chore/update-rafa-0.8.13
---

# Verifiable report — citation-linked claims + a fact-check graph node

## Why

Today `report` (`agents/python/src/lib/state.py:37`) is a plain string the model
writes via the `WriteReport` tool and the user free-edits in a `<Textarea>`
(`ResearchCanvas.tsx:191-200`). `resources` are fetched, cached, and rendered as
cards fully decoupled from the report text (`Resources.tsx`) — there is no
mechanism, today, connecting any sentence in the report to the resource(s) that
support it. The ask is two things: (1) every claim in the report should be
traceable to a resource, (2) a graph node should check the draft against the
sources and flag statements nothing supports.

## Approach

- **State**: add a `citations: List[Citation]` field to `AgentState`
  (`Citation{claim, resource_urls, supported, note}`), Python + frontend type
  only — see [verifiable-report-state](verifiable-report-state.md).
- **Backend**: a new chat-triggered tool `FactCheckReport` routes to a new
  `fact_check_node`, following the existing `search_node` structured-extraction
  pattern (forced `tool_choice` + a pydantic schema) — see
  [verifiable-report-factcheck-node](verifiable-report-factcheck-node.md).
- **Frontend**: a new decoupled panel (modeled on `Progress.tsx`'s step list,
  not an inline rewrite of the plain-text `Textarea`) renders each claim with a
  supported/unsupported badge and a clickable link to its backing resource(s)
  — see [verifiable-report-frontend-ui](verifiable-report-frontend-ui.md).
- Two open ledger items sit directly in these same files and are folded in as
  while-you're-here tasks: [verifiable-report-frontend-typing](verifiable-report-frontend-typing.md)
  (adopts `frontend-state-any-typing`) and
  [verifiable-report-initialstate](verifiable-report-initialstate.md) (adopts
  `coagent-initialstate-divergence`).
- [verifiable-report-verify](verifiable-report-verify.md) is a manual E2E pass
  — this repo has no automated test suite (no test script, no pytest config),
  so verification is a documented manual flow, not a claimed automated one.

## Non-goals (decided at approval, see Decisions)

- **No continuous/automatic fact-checking on every report edit.** Triggered
  on-demand via a chat request, exactly like `Search`/`DeleteResources` today —
  matches the app's existing all-actions-via-chat convention and avoids an LLM
  call on every keystroke.
- **No TypeScript agent parity in this pass.** Python stays canonical; per the
  brain's own convention the TS port ([agent-typescript-parity](../../brain/rules/agent-typescript-parity.md))
  is an accepted-lagging alternate not run by default. Deferred as a fast-follow.
- **Not restructuring `report` into a structured claim tree.** It stays the
  same free-text, user-editable field. Citations are a derived, decoupled
  layer (same shape of decision the app already made for Resources vs. report)
  — recomputed each time fact-check runs, so they can go stale between runs.
  Accepted; not solved here.

## Alternatives considered

- **Inline citation markers in the report text** (e.g. `[1]`) — rejected: no
  markdown renderer exists on the frontend today (plain `<Textarea>`), and
  markers would drift the moment the user edits the draft.
- **Automatic fact-check after every `WriteReport` call** — rejected: costs an
  extra LLM call per edit and would make editing feel sluggish; on-demand
  matches how every other capability in this app is user-initiated.

## Risks

- Claim segmentation is left to the model inside the forced-tool-choice schema
  (no hand-rolled sentence splitter), consistent with `ExtractResources`, but
  claim boundaries may not always match a user's intuitive reading of the draft.
- A report edited after a fact-check run has no automatic re-check — citations
  reflect whatever the report said at the last `FactCheckReport` call.

## Decisions

- **TS parity**: Python-only for now (TS parity deferred as a fast-follow) —
  matches the repo's existing accepted convention that the TS port lags and
  isn't run by default; halves the surface to build/test.
- **Trigger mechanism**: on-demand via chat (new no-op `FactCheckReport` tool,
  same pattern as `Search`/`DeleteResources`) — zero new UI, avoids an extra
  LLM call + latency on every report edit.

## Plan-done review gate

`rafa review`'s deterministic pre-pass (8 changed files, 31/32 notes engaged, 6
deep) + a prism-style judge pass found the diff coherent and grounded, and
caught 4 brain notes this diff had made stale/incomplete — fixed as branch
working-set edits (not touching main): `agent-state-shape-contract` (now
documents 6 fields, `citations` Python+frontend-only), `langgraph-agent-convention`
(now documents the 6th node, `fact_check_node`), `research-chat-flow` (now
documents the `FactCheckReport` branch + fixed 5 drifted line-cites), and
`model-selection-flow` (fixed one drifted line-cite). `rafa verify-citations`
passes 43/43 resolution, 1/1 policy, after the fixes. These land in the org
brain at merge-to-main distillation.

**Residual risk (surfaced, not silently accepted): no live, LLM-driven
fact-check run has ever executed** — this dev environment has no OpenAI/
Anthropic/Tavily API keys. Everything about the runtime path (model call →
citations populate → UI flags unsupported claims → next chat turn doesn't
400) is proven by static trace + parity with the already-working `search_node`,
not by execution. Run one live smoke test with real API keys before this
reaches real users.
