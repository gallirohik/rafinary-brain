---
type: "Citation Check"
title: "Citation check report"
description: "checker v2 — all gates pass · 4 warn(s)"
timestamp: 2026-07-31T22:26:43.915Z
---
# Citation check (generated — do not hand-edit) · checker v2

## Resolution (B1): 7/7 ✓
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

## Absence (B3, declared `absent:` re-grepped): 0/0 ✓

## Inventory (coverage declared vs `git ls-files`): 0/0 ✓

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 4
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → langgraph-agent-convention does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → agent-typescript-parity does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — frontmatter → research-chat-flow does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/resource-text-dominates-the-system-prompt.md — mdlink → /brain/rules/build-tooling-convention.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)

**All pass.**
