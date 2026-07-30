---
type: "Citation Check"
title: "Citation check report"
description: "checker v2 — all gates pass · 0 warn(s)"
timestamp: 2026-07-30T08:12:59.409Z
---
# Citation check (generated — do not hand-edit) · checker v2

## Resolution (B1): 53/53 ✓
✓ src/app/globals.css:6 :: Arial
✓ src/app/layout.tsx:6 :: localFont
✓ package.json:19 :: @copilotkit/runtime
✓ pnpm-lock.yaml:3669 :: uuid@10.0.0
✓ pnpm-lock.yaml:438 :: '@hono/node-server@1.19.15'
✓ src/app/api/copilotkit/route.ts:7 :: @copilotkit/runtime
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
✓ src/app/api/copilotkit/route.ts:92 :: lgcDeploymentUrl
✓ src/app/api/copilotkit/route.ts:19 :: LANGSMITH_API_KEY
✓ src/app/api/copilotkit/route.ts:122 :: deploymentUrl
✓ src/app/api/copilotkit/route.ts:123 :: langsmithApiKey
✓ src/app/api/copilotkit/route.ts:81 :: addresses.every
✓ src/app/api/copilotkit/route.ts:22 :: visible to anyone via devtools
✓ src/lib/model-selector-provider.tsx:40 :: lgcDeploymentUrl
✓ src/app/page.tsx:24 :: lgcDeploymentUrl
✓ package.json:27 :: next
✓ pnpm-lock.yaml:2816 :: next@15.5.15
✓ src/app/api/copilotkit/route.ts:86 :: export const POST
✓ pnpm-lock.yaml:3045 :: postcss@8.4.31
✓ pnpm-lock.yaml:3349 :: sharp@0.34.5
✓ package.json:39 :: postcss
✓ postcss.config.mjs:4 :: tailwindcss
✓ agents/python/src/lib/chat.py:79 :: ChatOpenAI
✓ agents/python/src/lib/chat.py:80 :: parallel_tool_calls
✓ agents/python/src/lib/chat.py:120 :: tool_calls[0]
✓ agents/python/src/lib/chat.py:129 :: tool_calls[0]
✓ agents/python/src/lib/chat.py:153 :: tool_calls[0]
✓ agents/python/src/lib/model.py:30 :: ChatAnthropic
✓ agents/python/src/lib/model.py:39 :: ChatGoogleGenerativeAI
✓ agents/python/src/lib/chat.py:109 :: {resources}
✓ agents/python/src/lib/chat.py:74 :: resources.append
✓ agents/python/src/lib/download.py:78 :: _RESOURCE_CACHE
✓ agents/typescript/src/chat.ts:21 :: MAX_TOTAL_RESOURCE_CHARS
✓ agents/typescript/src/download.ts:17 :: MAX_RESOURCE_CHARS
✓ package.json:11 :: dev:agent:py
✓ agents/python/src/lib/download.py:17 :: _RESOURCE_CACHE
✓ agents/python/src/lib/model.py:32 :: claude-3-5-sonnet-20240620
✓ agents/python/src/lib/model.py:41 :: gemini-1.5-pro
✓ agents/typescript/src/model.ts:26 :: claude-3-5-sonnet-20240620
✓ agents/typescript/src/model.ts:32 :: gemini-1.5-pro
✓ src/app/Main.tsx:49 :: setState
✓ src/app/Main.tsx:50 :: setTimeout

## Completeness (B2): 0/0 ✓  (0 anchors)

## Policy (contract → anchor declared): 0/0 ✓

## Absence (B3, declared `absent:` re-grepped): 0/0 ✓

## Inventory (coverage declared vs `git ls-files`): 0/0 ✓

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 0

**All pass.**
