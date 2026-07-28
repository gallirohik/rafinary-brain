---
type: "Citation Check"
title: "Citation check report"
description: "checker v2 — all gates pass · 10 warn(s)"
timestamp: 2026-07-28T14:21:21.077Z
---
# Citation check (generated — do not hand-edit) · checker v2

## Resolution (B1): 26/26 ✓
✓ package.json:9 :: concurrently
✓ package.json:12 :: "dev:agent:py"
✓ package.json:14 :: install:agent:ts
✓ package.json:17 :: "@copilotkit/react-core"
✓ package.json:43 :: "nx"
✓ agents/typescript/package.json:13 :: "@copilotkit/sdk-js"
✓ agents/typescript/package.json:23 :: "langchain"
✓ agents/python/langgraph.json:5 :: graphs
✓ agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
✓ vercel.json:2 :: nx run
✓ next.config.mjs:3 :: standalone
✓ agents/python/main.py:41 :: PORT
✓ agents/python/uv.lock:1 :: version
✓ package.json:13 :: uv sync
✓ readme.md:24 :: cd agent-py
✓ readme.md:25 :: poetry install
✓ readme.md:60 :: cd ./ui
✓ readme.md:77 :: remoteEndpoints
✓ readme.md:108 :: ./agent-py
✓ agents/typescript/src/chat.ts:114 :: JSON.stringify(resources)
✓ agents/typescript/src/chat.ts:21 :: MAX_TOTAL_RESOURCE_CHARS
✓ agents/typescript/src/chat.ts:73 :: MAX_TOTAL_RESOURCE_CHARS
✓ agents/typescript/src/download.ts:17 :: MAX_RESOURCE_CHARS
✓ agents/python/src/lib/chat.py:109 :: {resources}
✓ agents/python/src/lib/chat.py:74 :: resources.append
✓ agents/python/src/lib/download.py:78 :: _RESOURCE_CACHE

## Completeness (B2): 2/2 ✓  (1 anchors)
✓ anchor 'MAX_TOTAL_RESOURCE_CHARS' → agents/typescript/src/chat.ts:21
✓ anchor 'MAX_TOTAL_RESOURCE_CHARS' → agents/typescript/src/chat.ts:73

## Policy (contract → anchor declared): 1/1 ✓
✓ rules/resource-text-dominates-the-system-prompt.md

## Absence (B3, declared `absent:` re-grepped): 1/1 ✓
✓ absent 'workspace:*' → (nowhere — as claimed)

## Inventory (coverage declared vs `git ls-files`): 0/0 ✓

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 10
⚠ rules/build-tooling-convention.md — frontmatter → langgraph-agent-convention does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — frontmatter → agent-typescript-parity does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — frontmatter → env-and-integrations does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — mdlink → /brain/rules/agent-typescript-parity.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — mdlink → /brain/rules/agent-typescript-parity.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — mdlink → /brain/rules/env-and-integrations.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/build-tooling-convention.md — mdlink → /brain/rules/copilotkit-runtime-route-convention.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → langgraph-agent-convention does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → agent-typescript-parity does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → research-chat-flow does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)

**All pass.**
