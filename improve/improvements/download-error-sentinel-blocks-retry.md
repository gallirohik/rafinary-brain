---
schemaVersion: 1
id: download-error-sentinel-blocks-retry
priority: P1
category: correctness
status: open
title: The "ERROR" cache sentinel is truthy, so a URL that fails to download once is never retried
summary: _download_resource writes the literal string "ERROR" into _RESOURCE_CACHE on any failure, but download_node's re-download test is `if not get_resource(url)` — "ERROR" is a non-empty (truthy) string, so one transient network blip permanently poisons that resource for the life of the process while the UI reports the download as done
fix: Test for real content in download_node — `if get_resource(url) in ("", "ERROR")` — so errored URLs are retried instead of being cached as permanent failures (~10 min)
leverage: { impact: high, effort: low }
blast_radius: [agent-python, data-persistence]
cites:
  - agents/python/src/lib/download.py:97 :: if not get_resource
  - agents/python/src/lib/download.py:24 :: _RESOURCE_CACHE.get
  - agents/python/src/lib/download.py:66 :: "ERROR"
  - agents/python/src/lib/download.py:81 :: "ERROR"
  - agents/python/src/lib/chat.py:72 :: content == "ERROR"
  - agents/python/src/lib/fact_check.py:81 :: ("", "ERROR")
found: 2026-07-27
---
`_download_resource` uses the literal string `"ERROR"` as a failure sentinel in
`_RESOURCE_CACHE` — written on SSRF rejection (`download.py:66`) and on **any** exception at
all via a bare `except Exception` (`download.py:81`), which includes a transient DNS hiccup,
a timeout, or a one-off 5xx. Three consumers branch on that sentinel, and one of them is
wrong ([langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)):

| site | test | correct? |
| --- | --- | --- |
| `chat_node` | `if content == "ERROR": continue` (`chat.py:72`) | ✓ skips it |
| `fact_check_node` | `if content in ("", "ERROR"): continue` (`fact_check.py:81`) | ✓ skips it |
| `download_node` | `if not get_resource(resource["url"])` (`download.py:97`) | ✗ **truthiness bug** |

`get_resource` returns `_RESOURCE_CACHE.get(url, "")` (`download.py:24`). Only `""` — the
never-fetched case — is falsy. `"ERROR"` is a non-empty string and therefore **truthy**, so
`not get_resource(url)` is `False` and the URL is never re-added to `resources_to_download`.
**A URL that fails once is never retried for the life of the process.**

## Why this is the silent kind

Nothing raises. The user-visible sequence is: the resource card stays on the canvas, the
progress log for that URL is marked `done` (`download.py:109` flips `done` for every entry it
queued — and after the first failure it queues nothing at all, so the log stays clean), the
model never receives the content, and no error reaches `state` or the UI. Two downstream
symptoms a dev will actually report:

- *"my resource has no content"* — `chat_node` silently drops it from `resources`.
- *"fact-check says nothing supports this claim"* — `fact_check_node` skips the same resource,
  so a genuinely-supported claim comes back `supported: false` with an empty `resource_urls`.
  This is a **correctness** failure in the report itself, not merely a missing fetch.

The only recovery today is restarting the process (the cache is module-level and dies with it).

## The fix, and the trap in it

Change the **test**, not just the sentinel: `if get_resource(resource["url"]) in ("", "ERROR")`.
Swapping `"ERROR"` for `None` without touching `download.py:97` would work too, but then the
two `== "ERROR"` consumers (`chat.py:72`, `fact_check.py:81`) both go stale and start feeding
the model the literal text of the sentinel — change all four sites together or change only the
truthiness test. Note also that any future eviction policy for the cache
(see `resource-cache-unbounded`) must treat a miss as "re-download", never as "unsupported
claim" — the same asymmetry, one layer down.

Related but distinct: `resource-cache-unbounded` (P3) tracks the cache's *unbounded growth*.
This row is about *permanent failure caching* — different defect, same dict.
