---
schemaVersion: 1
id: resource-cache-unbounded
priority: P3
lens: performance
status: open
title: _RESOURCE_CACHE grows unbounded for the life of the process
summary: Downloaded page content is memoized in a module-level dict that is never evicted or size-capped, so a long-running agent process accumulates every fetched URL's full markdown in memory
fix: Cap the cache (LRU / max entries) or add a size/TTL bound in download.py (~15 min)
leverage: { impact: low, effort: medium }
blast_radius: [agent-python, data-persistence]
cites:
  - agents/python/src/lib/download.py:17 :: _RESOURCE_CACHE
found: 2026-07-20
---
`_RESOURCE_CACHE` (`download.py:17`) memoizes each fetched URL's full HTML-to-markdown body
keyed by URL, and is written on every successful (or errored) download (`download.py:78,81`).
Nothing ever evicts it, so a process serving many research sessions grows in memory unbounded —
the brain documents it as an in-process cache that "dies with the process," which is fine for a
demo but is a slow leak under sustained use. Bound it (LRU with a max entry count, or a TTL) if
this ever runs as a persistent service. Low priority for the example app; noted so the assumption
is explicit.
