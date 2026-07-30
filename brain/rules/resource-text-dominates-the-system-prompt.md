---
schemaVersion: 1
id: resource-text-dominates-the-system-prompt
type: contract
domain: agent-bridge
title: Every resource's full page text is re-sent in the system prompt on every turn — capped in TypeScript, still uncapped in Python
summary: Both chat nodes re-serialise all downloaded resources into the system prompt each turn, so one long article can exceed an entire per-minute token budget in a single request; the TS agent caps this (8k/resource, 24k total) as of 2026-07-28, the Python agent does NOT
links: [langgraph-agent-convention, agent-typescript-parity, research-chat-flow]
failure: loud
anchor: MAX_TOTAL_RESOURCE_CHARS
cites:
  - agents/typescript/src/chat.ts:114 :: JSON.stringify(resources)
  - agents/typescript/src/chat.ts:21 :: MAX_TOTAL_RESOURCE_CHARS
  - agents/typescript/src/chat.ts:73 :: MAX_TOTAL_RESOURCE_CHARS
  - agents/typescript/src/download.ts:17 :: MAX_RESOURCE_CHARS
  - agents/python/src/lib/chat.py:109 :: {resources}
  - agents/python/src/lib/chat.py:74 :: resources.append
  - agents/python/src/lib/download.py:78 :: _RESOURCE_CACHE
description: "Both chat nodes re-serialise all downloaded resources into the system prompt each turn, so one long article can exceed an entire per-minute token budget in a single request; the TS agent caps this (8k/resource, 24k total) as of 2026-07-28, the Python agent does NOT"
tags: [agent-bridge]
timestamp: 2026-07-28T14:39:04Z
---

The research agents do not summarise their sources. Each `chat_node` walks
`state["resources"]`, pulls the **entire** cached page text out of the download
cache, and serialises the whole set into the system prompt — on **every turn**,
not just the turn the resource was added.

That makes prompt size a function of *how much has been read*, not of how long
the conversation is. It is the dominant term in this app's token cost by a wide
margin.

## Why it fails loudly and unrecoverably

Measured 2026-07-28 with `html-to-text` on real pages:

| page | raw | as tokens |
| --- | --- | --- |
| en.wikipedia.org/wiki/France | 458,781 chars | ~124,000 |
| en.wikipedia.org/wiki/Paris | 426,978 chars | ~115,000 |

A single Wikipedia article therefore exceeds a 30k TPM budget **on its own**.
That surfaces as:

```
429 Request too large for gpt-4o ... Limit 30000, Requested 124252
```

Read that error carefully — it is *not* throttling. When one request is larger
than the whole per-minute allowance, waiting and retrying can never succeed.
Anything that looks like "the agent 429s on the first message with one resource
attached" is this, not rate pressure.

## TypeScript: capped

Two caps, because they fail differently:

- `MAX_RESOURCE_CHARS = 8000` at download time (`download.ts:17`), with a
  `[truncated]` marker appended so the model knows the source is partial.
- `MAX_TOTAL_RESOURCE_CHARS = 24000` as an aggregate budget at injection time
  (`chat.ts:21`, consumed at `:73`). Per-resource capping alone still scales
  with resource *count* — ten capped pages would rebuild the same problem.

## Python: NOT capped — and Python is the default

`agents/python` has the identical shape with no limit: `download.py:78` caches
`html2text` output whole, and `chat.py:74` appends full `content` into the list
that `chat.py:109` interpolates into the prompt.

This matters more than the TS case, not less: `npm run dev` starts the **Python**
agent ([build-tooling](/brain/rules/build-tooling-convention.md)), and the
recorded `verifiable-report` decision is "Python-only for now". So the agent
most people actually run is the uncapped one. Treat the TS caps as the
reference implementation to port, not as the fix.

## Not the only unbounded term

`report` and the accumulated `state.messages` are also injected in full each
turn. Resources are the dominant term, not the sole one — if prompt size still
grows after capping resources, look there next.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/typescript/src/chat.ts:114](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/chat.ts#L114) — `JSON.stringify(resources)`
[2] [agents/typescript/src/chat.ts:21](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/chat.ts#L21) — `MAX_TOTAL_RESOURCE_CHARS`
[3] [agents/typescript/src/chat.ts:73](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/chat.ts#L73) — `MAX_TOTAL_RESOURCE_CHARS`
[4] [agents/typescript/src/download.ts:17](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/download.ts#L17) — `MAX_RESOURCE_CHARS`
[5] [agents/python/src/lib/chat.py:109](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/chat.py#L109) — `{resources}`
[6] [agents/python/src/lib/chat.py:74](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/chat.py#L74) — `resources.append`
[7] [agents/python/src/lib/download.py:78](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L78) — `_RESOURCE_CACHE`

<!-- okf:citations:end -->
