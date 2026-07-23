---
schemaVersion: 1
id: verifiable-report-frontend-ui
plan: verifiable-report
parent: verifiable-report
kind: task
title: Render citations in the canvas — claim list linked to resources, unsupported claims flagged
description: >-
  The "every claim links to a resource" + "flags unsupported statements" half
  of the ask, on the frontend. Resources already render fully decoupled from
  the report (Resources.tsx cards); citations get a similar decoupled panel
  rather than an inline rewrite of the plain-text Textarea (which can't render
  links or highlights).
approach: >-
  New src/components/FactCheck.tsx modeled on Progress.tsx's step-list
  pattern: renders state.citations, one row per claim, a supported/unsupported
  badge, and clickable chips linking to the backing resource_urls (reusing the
  existing resource-card link behavior). Wire it into ResearchCanvas.tsx next
  to the existing Resources panel.
status: todo
priority: 2
blocked_by: [verifiable-report-state, verifiable-report-frontend-typing]
---

## Done-check

- `src/components/FactCheck.tsx` renders one row per `state.citations` entry
  with a visually distinct supported vs. unsupported state, and each cited
  `resource_urls` entry is a clickable link to that resource.
- `ResearchCanvas.tsx` renders `FactCheck` when `state.citations` is non-empty.
- `npm run build` and `npm run lint` pass.
- Manual check (paired with the fact-check node task's manual run): after
  triggering fact-check in chat, the claim list appears, the unsupported claim
  is visually flagged, and clicking a citation's resource link takes you to
  that resource.
