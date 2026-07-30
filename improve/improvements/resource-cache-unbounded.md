---
schemaVersion: 1
id: resource-cache-unbounded
priority: P3
category: performance
status: open
title: _RESOURCE_CACHE grows unbounded for the life of the process
summary: Downloaded page content is memoized in a module-level dict that is never evicted or size-capped, so a long-running agent process accumulates every fetched URL's full markdown in memory
fix: Cap the cache (LRU / max entries) or add a size/TTL bound in download.py (~15 min)
leverage: { impact: low, effort: medium }
blast_radius: [agent-python, data-persistence]
cites:
  - agents/python/src/lib/download.py:17 :: _RESOURCE_CACHE
found: 2026-07-20
type: Improvement
description: "Downloaded page content is memoized in a module-level dict that is never evicted or size-capped, so a long-running agent process accumulates every fetched URL's full markdown in memory"
timestamp: 2026-07-20
tags: [performance, P3]
---
`_RESOURCE_CACHE` (`download.py:17`) memoizes each fetched URL's full HTML-to-markdown body
keyed by URL, and is written on every successful (or errored) download (`download.py:66,78,81`).
Nothing ever evicts it, so a process serving many research sessions grows in memory unbounded —
the brain documents it as an in-process cache that "dies with the process," which is fine for a
demo but is a slow leak under sustained use. Bound it (LRU with a max entry count, or a TTL) if
this ever runs as a persistent service. Low priority for the example app; noted so the assumption
is explicit.

Second reader since the 2026-07-27 refresh: `fact_check_node` now reads the same cache via
`get_resource` rather than re-fetching, so the cache is load-bearing for correctness of the
fact-check pass, not just a speed-up. Any eviction policy must not silently drop a resource the
report still cites — evict by size/age, and treat a miss as "re-download", never as "unsupported
claim".

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/download.py:17](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L17) — `_RESOURCE_CACHE`

<!-- okf:citations:end -->
