---
id: python-resource-text-uncapped
type: Improvement
schemaVersion: 1
priority: P1
category: performance
status: open
title: The TypeScript agent caps resource text in the system prompt; the Python agent — the one that actually runs — does not
summary: "chat_node serialises every downloaded resource's full page text into the system prompt on every turn with no truncation and no aggregate budget, so one long article can exceed an entire per-minute token allowance in a single request and 429 unrecoverably; the TS port fixed exactly this with MAX_RESOURCE_CHARS/MAX_TOTAL_RESOURCE_CHARS, but npm run dev starts the Python backend"
fix: "Port the TS caps to Python — truncate each cached page to ~8000 chars with a [truncated] marker in download.py, and budget the aggregate to ~24000 chars in chat_node's resource loop (~20 min)"
leverage: { impact: high, effort: low }
blast_radius: [agent-python, agent-bridge, data-persistence]
cites:
  - agents/python/src/lib/chat.py:109 :: {resources}
  - agents/python/src/lib/chat.py:74 :: resources.append
  - agents/python/src/lib/download.py:78 :: _RESOURCE_CACHE
  - agents/typescript/src/chat.ts:21 :: MAX_TOTAL_RESOURCE_CHARS
  - agents/typescript/src/download.ts:17 :: MAX_RESOURCE_CHARS
  - package.json:11 :: dev:agent:py
found: 2026-07-30
description: "chat_node serialises every downloaded resource's full page text into the system prompt on every turn with no truncation and no aggregate budget, so one long article can exceed an entire per-minute token allowance in a single request and 429 unrecoverably; the TS port fixed exactly this with MAX_RESOURCE_CHARS/MAX_TOTAL_RESOURCE_CHARS, but npm run dev starts the Python backend"
tags: [performance, P1]
timestamp: 2026-07-30
---
The brain already documents this as a live asymmetry
([resource-text-dominates-the-system-prompt](/brain/rules/resource-text-dominates-the-system-prompt.md)):
*"the TS agent caps this (8k/resource, 24k total) as of 2026-07-28, the Python agent does
NOT."* What the ledger was missing is the row — the knowledge existed, the debt was not
tracked, so it could never surface at plan or build time in this region.

## Why this is P1 rather than a nit

The uncapped half is **the half that runs**. `npm run dev` starts `dev:agent`, which is
`npm run dev:agent:py` (`package.json:11`) — the TypeScript agent is the accepted-lagging
port. So the fix landed on the backend nobody is exercising, and the default path still
carries the defect.

The failure is not gradual. `chat_node` walks `state["resources"]`, pulls each entry's whole
cached body out of the download cache (`chat.py:74`, cache written at `download.py:78`) and
interpolates the lot into the system prompt (`chat.py:109`) — on **every turn**, not just the
turn a resource was added. Prompt size therefore tracks how much has been *read*, not how long
the conversation is. The brain's measurement, taken on real pages: a single Wikipedia article
runs ~115–124k tokens, which on a 30k TPM budget produces

```
429 Request too large for gpt-4o ... Limit 30000, Requested 124252
```

That is not throttling — a request larger than the entire per-minute allowance can never
succeed, so waiting and retrying never clears it. Symptom to recognise: *"the agent 429s on
the first message once a resource is attached."*

## The fix is a port, not a design

Both constants already exist and are already tuned, with the reasoning written down next to
them:

- `MAX_RESOURCE_CHARS = 8000` at download time (`download.ts:17`), with a `[truncated]`
  marker appended so the model knows the source is partial.
- `MAX_TOTAL_RESOURCE_CHARS = 24000` as an aggregate budget at injection time
  (`chat.ts:21`) — per-resource truncation alone still scales with resource count.

Mirror both in `download.py` and `chat_node`'s resource loop. Two caps rather than one,
because they fail differently: the per-resource cap bounds a single huge page, the aggregate
cap bounds many medium ones. Keep the `[truncated]` marker — a silently shortened source is
worse than a declared one, since the model will cite it as if complete.

While in `chat_node`, note the loop already skips `"ERROR"` entries (`chat.py:72`); the
truncation belongs alongside that filter, not in a second pass.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/chat.py:109](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/chat.py#L109) — `{resources}`
[2] [agents/python/src/lib/chat.py:74](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/chat.py#L74) — `resources.append`
[3] [agents/python/src/lib/download.py:78](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L78) — `_RESOURCE_CACHE`
[4] [agents/typescript/src/chat.ts:21](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/chat.ts#L21) — `MAX_TOTAL_RESOURCE_CHARS`
[5] [agents/typescript/src/download.ts:17](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/download.ts#L17) — `MAX_RESOURCE_CHARS`
[6] [package.json:11](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/package.json#L11) — `dev:agent:py`

<!-- okf:citations:end -->
