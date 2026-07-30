---
id: state-persistence-convention
type: convention
schemaVersion: 1
domain: data-persistence
title: "Nothing is persisted — resource text and graph checkpoints live in process memory, and the sqlite checkpointer deps are never used"
summary: "There is no database, ORM, or client-side storage anywhere in this app: downloaded page text sits in a module-level _RESOURCE_CACHE dict and conversation state in an in-memory MemorySaver, so a backend restart silently empties both; langgraph-checkpoint-sqlite/aiosqlite are declared dependencies whose savers are never imported, and the TS agent compiles with no checkpointer at all"
links: [langgraph-agent-convention, agent-typescript-parity, agent-state-shape-contract, research-chat-flow, resource-text-dominates-the-system-prompt]
anchor: _RESOURCE_CACHE
absent: SqliteSaver
absent: localStorage
absent: prisma
absent: drizzle
cites:
  - agents/python/src/lib/download.py:17 :: _RESOURCE_CACHE
  - agents/python/src/lib/download.py:24 :: _RESOURCE_CACHE
  - agents/python/src/lib/download.py:66 :: _RESOURCE_CACHE
  - agents/python/src/lib/download.py:78 :: _RESOURCE_CACHE
  - agents/python/src/lib/download.py:81 :: _RESOURCE_CACHE
  - agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
  - agents/python/src/agent.py:45 :: MemorySaver
  - agents/python/src/agent.py:47 :: MemorySaver
  - agents/python/src/agent.py:48 :: checkpointer
  - agents/python/main.py:12 :: LANGGRAPH_FASTAPI
  - agents/python/pyproject.toml:24 :: langgraph-checkpoint-sqlite
  - agents/python/pyproject.toml:25 :: aiosqlite
  - agents/typescript/src/agent.ts:8 :: MemorySaver
  - agents/typescript/src/agent.ts:33 :: workflow.compile
  - agents/python/src/lib/fact_check.py:80 :: get_resource
  - src/components/ResearchCanvas.tsx:87 :: setState
description: "There is no database, ORM, or client-side storage anywhere in this app: downloaded page text sits in a module-level _RESOURCE_CACHE dict and conversation state in an in-memory MemorySaver, so a backend restart silently empties both; langgraph-checkpoint-sqlite/aiosqlite are declared dependencies whose savers are never imported, and the TS agent compiles with no checkpointer at all"
tags: [data-persistence]
timestamp: 2026-07-30T07:03:57.993Z
---
Read this before you plan anything that assumes stored data — "cache the resource",
"resume the session", "show the user's past reports". **This app has no persistence layer
at all.** Every byte lives in the backend process, and dies with it.

**There is no database and no client storage.** Not "not yet configured" — nowhere in
`src/` or `agents/` is there an ORM, a driver, or a browser store. The `absent:` tokens in
this note's frontmatter (`prisma`, `drizzle`, `SqliteSaver`, `localStorage`) are re-grepped
on every check, so this claim cannot silently go stale: the day one of them appears, the
gate fails and this note is wrong on purpose. Nothing is written to disk, no user is
identified, and no state survives a process restart — which is also why
[security-posture](/brain/playbooks/security-posture.md) finds no auth chokepoint to guard.

**The two things that ARE held, both in RAM:**

1. **Downloaded page text — `_RESOURCE_CACHE`, a module-level dict** (`download.py:17`).
   It is the *only* store of fetched resource content. `get_resource(url)` reads it
   (`:24`), `_download_resource` writes either the html2text markdown (`:78`) or the
   literal string `"ERROR"` on a rejected/failed fetch (`:66`, `:81`). Consequences that
   bite:
   - It is **keyed by URL and never evicted or bounded** — see the open `resource-cache-unbounded`
     improvement. Long sessions grow it until the process is recycled.
   - `"ERROR"` is a **sticky sentinel**: once a URL fails, that string is cached, and
     `chat_node`/`fact_check_node` skip the resource forever rather than retrying (the open
     `download-error-sentinel-blocks-retry` improvement). A transient network blip
     permanently drops a resource for the life of the process.
   - `fact_check_node` reads this same cache rather than re-fetching (`fact_check.py:80`),
     so fact-check accuracy depends on whatever `download_node` happened to capture
     earlier — a resource added but never successfully downloaded is invisible to it.
   - Because the cache is **per-process and module-level**, it is shared across all
     conversations and all users of that backend, and it is *not* part of `AgentState`.
     Two browser tabs hitting the same server share it.

2. **Conversation/graph state — an in-memory `MemorySaver` checkpointer** (`agent.py:45,47`,
   attached at `:48`). This is what makes the human-in-the-loop delete interrupt resumable.
   The choice is **conditional on an env var**: `agent.py:40` uses `MemorySaver` only when
   `LANGGRAPH_FASTAPI` is *not* `"false"`, and `main.py:12` sets it to `"true"` before
   importing the graph — so the local FastAPI server always gets `MemorySaver`. Under
   LangGraph Cloud the platform supplies its own (durable) checkpointer instead, so
   **resumability differs between local dev and a deployed graph** and local behaviour is
   the weaker of the two.

**The trap — declared sqlite deps that nothing imports.** `pyproject.toml:24-25` pull in
`langgraph-checkpoint-sqlite` and `aiosqlite`. A reasonable dev greps the dependency list,
concludes checkpoints are on sqlite, and plans accordingly. They are not: `SqliteSaver` /
`AsyncSqliteSaver` are never imported anywhere (declared `absent:` above, so the gate
proves it every run). The dependencies are inherited from the upstream example. If you
*want* durable checkpoints locally, the deps are already there — but wiring them is real
work in `agent.py:40-48`, not a config flip.

**The TypeScript agent has no checkpointer at all.** `agent.ts:8` imports `MemorySaver`
and then never uses it — `workflow.compile({ interruptAfter: ["delete_node"] })`
(`agent.ts:33`) passes no `checkpointer`. So the delete interrupt cannot resume there. One
more entry for the drift list in
[agent-typescript-parity](/brain/rules/agent-typescript-parity.md).

**What the frontend "persists": nothing.** Resource edits go through `setState` on the
coagent (`ResearchCanvas.tsx:87`), which round-trips into
[AgentState](/brain/rules/agent-state-shape-contract.md) — the graph's in-memory state.
Reload the page and the canvas is empty. The `?coAgentsModel=` URL param is the only thing
that survives a reload, and that is a query string, not storage
([model-selection-flow](/brain/playbooks/model-selection-flow.md)).

So: to add anything durable — saved reports, history, multi-user separation — you are
adding the persistence layer, not extending one. Budget for it.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/download.py:17](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L17) — `_RESOURCE_CACHE`
[2] [agents/python/src/lib/download.py:24](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L24) — `_RESOURCE_CACHE`
[3] [agents/python/src/lib/download.py:66](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L66) — `_RESOURCE_CACHE`
[4] [agents/python/src/lib/download.py:78](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L78) — `_RESOURCE_CACHE`
[5] [agents/python/src/lib/download.py:81](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L81) — `_RESOURCE_CACHE`
[6] [agents/python/src/agent.py:40](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/agent.py#L40) — `LANGGRAPH_FASTAPI`
[7] [agents/python/src/agent.py:45](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/agent.py#L45) — `MemorySaver`
[8] [agents/python/src/agent.py:47](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/agent.py#L47) — `MemorySaver`
[9] [agents/python/src/agent.py:48](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/agent.py#L48) — `checkpointer`
[10] [agents/python/main.py:12](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L12) — `LANGGRAPH_FASTAPI`
[11] [agents/python/pyproject.toml:24](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/pyproject.toml#L24) — `langgraph-checkpoint-sqlite`
[12] [agents/python/pyproject.toml:25](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/pyproject.toml#L25) — `aiosqlite`
[13] [agents/typescript/src/agent.ts:8](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/agent.ts#L8) — `MemorySaver`
[14] [agents/typescript/src/agent.ts:33](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/agent.ts#L33) — `workflow.compile`
[15] [agents/python/src/lib/fact_check.py:80](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/fact_check.py#L80) — `get_resource`
[16] [src/components/ResearchCanvas.tsx:87](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/components/ResearchCanvas.tsx#L87) — `setState`

<!-- okf:citations:end -->
