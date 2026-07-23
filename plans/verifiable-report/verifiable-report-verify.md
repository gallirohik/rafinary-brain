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
status: todo
priority: 2
blocked_by: [verifiable-report-factcheck-node, verifiable-report-frontend-ui, verifiable-report-initialstate]
---

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
