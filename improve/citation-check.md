---
type: "Citation Check"
title: "Citation check report"
description: "checker v2 — all gates pass · 0 warn(s)"
timestamp: 2026-07-26T21:16:19.978Z
---
# Citation check (generated — do not hand-edit) · checker v2

## Resolution (B1): 47/47 ✓
✓ src/app/globals.css:6 :: Arial
✓ src/app/layout.tsx:6 :: localFont
✓ src/app/Main.tsx:12 :: initialState
✓ src/components/ResearchCanvas.tsx:24 :: initialState
✓ src/lib/types.ts:32 :: createInitialAgentState
✓ package.json:19 :: @copilotkit/runtime
✓ pnpm-lock.yaml:3669 :: uuid@10.0.0
✓ pnpm-lock.yaml:438 :: '@hono/node-server@1.19.15'
✓ src/app/api/copilotkit/route.ts:7 :: @copilotkit/runtime
✓ src/app/layout.tsx:18 :: Research Helper
✓ src/app/layout.tsx:19 :: AI-assisted research canvas
✓ agents/python/src/lib/download.py:97 :: if not get_resource
✓ agents/python/src/lib/download.py:24 :: _RESOURCE_CACHE.get
✓ agents/python/src/lib/download.py:66 :: "ERROR"
✓ agents/python/src/lib/download.py:81 :: "ERROR"
✓ agents/python/src/lib/chat.py:72 :: content == "ERROR"
✓ agents/python/src/lib/fact_check.py:81 :: ("", "ERROR")
✓ agents/python/src/lib/fact_check.py:65 :: tool_calls[0]
✓ agents/python/src/lib/fact_check.py:114 :: tool_calls[0]
✓ agents/python/src/lib/fact_check.py:129 :: tool_calls[0]
✓ agents/python/src/lib/fact_check.py:141 :: tool_calls[0]
✓ agents/python/src/lib/search.py:67 :: if not ai_message.tool_calls
✓ agents/python/src/lib/delete.py:26 :: if ai_message.tool_calls
✓ src/lib/types.ts:23 :: Resource[]
✓ src/lib/types.ts:24 :: Log[]
✓ src/lib/types.ts:14 :: Log
✓ src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
✓ src/app/api/copilotkit/route.ts:74 :: dns.lookup
✓ src/app/api/copilotkit/route.ts:29 :: isPrivateIPv4
✓ src/app/api/copilotkit/route.ts:43 :: isPrivateIPv6
✓ agents/python/src/lib/download.py:41 :: getaddrinfo
✓ package.json:27 :: next
✓ pnpm-lock.yaml:2816 :: next@15.5.15
✓ src/app/api/copilotkit/route.ts:86 :: export const POST
✓ pnpm-lock.yaml:3045 :: postcss@8.4.31
✓ pnpm-lock.yaml:3349 :: sharp@0.34.5
✓ package.json:39 :: postcss
✓ postcss.config.mjs:4 :: tailwindcss
✓ agents/python/src/lib/download.py:17 :: _RESOURCE_CACHE
✓ agents/python/src/lib/search.py:74 :: tool_calls[0]
✓ agents/python/src/lib/search.py:152 :: tool_calls[0]
✓ agents/python/src/lib/search.py:145 :: tool_call_id
✓ agents/typescript/src/search.ts:53 :: tool_calls?.length
✓ agents/python/src/lib/model.py:32 :: claude-3-5-sonnet-20240620
✓ agents/python/src/lib/model.py:41 :: gemini-1.5-pro
✓ src/app/Main.tsx:49 :: setState
✓ src/app/Main.tsx:50 :: setTimeout

## Completeness (B2): 0/0 ✓  (0 anchors)

## Policy (contract → anchor declared): 0/0 ✓

## Absence (B3, declared `absent:` re-grepped): 0/0 ✓

## Inventory (coverage declared vs `git ls-files`): 0/0 ✓

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 0

**All pass.**
