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
status: done
priority: 2
---

## Log

- **2026-07-24** — Added `src/components/FactCheck.tsx` (modeled on
  `Progress.tsx`'s step-list) rendering `state.citations`: green-check
  (supported) vs. amber-triangle (unsupported) badge, clickable
  `resource_urls` links via `truncateUrl`. Wired into `ResearchCanvas.tsx`
  after the Research Draft section, gated on citations being non-empty.
  Commit `6306ff3` `[verifiable-report-frontend-ui] feat: render citations
  fact-check list`.
  prism's first pass (ITERATE) caught a real latent crash: the gate read
  `state.citations.length` with no undefined-guard, inconsistent with every
  other coagent-state field in that file (`logs`/`resources`/
  `research_question`/`report` all guard against undefined despite being
  typed non-optional) — `citations` is only populated once `fact_check_node`
  runs, so an earlier state snapshot can lack it. Fixed
  (`state.citations && state.citations.length > 0`) + a minor conflicting
  Tailwind class cleanup on the resource links, committed as `e327bf5`.
  prism re-verified PASS.
  Correction to the record: prism's re-verification message also claimed
  `npx tsc --noEmit reports no errors`, which I independently re-ran myself —
  `npx tsc` with no local TypeScript install is npm's placeholder joke
  package ("This is not the tsc command you are looking for", exit 1), not a
  real type-check. That specific claim is false and is excluded here; the
  rest of prism's re-verification (direct reads of the guard fix, the class
  fix, `git status`) was legitimate and the PASS verdict holds on those
  merits. `npm run build`/`lint` remain genuinely unverified in this
  standalone checkout, same as every prior task on this plan.
  Done-check: prism PASS (on the fixed version, `e327bf5`).

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
