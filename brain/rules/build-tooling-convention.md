---
schemaVersion: 1
id: build-tooling-convention
type: convention
domain: build-tooling
title: "Dev/build wiring — concurrently runs UI + Python agent; the root app AND the TS agent are now de-workspaced, but vercel.json and readme.md still assume the upstream monorepo (readme.md is a non-exemplar — do not follow it)"
summary: "npm run dev launches Next.js (3000) and the Python agent (8000) together; the root CopilotKit deps and — as of 2026-07-28 — agents/typescript were both de-workspaced to real npm ranges, so each installs standalone; vercel.json still cd's three levels up to run nx and readme.md still documents the upstream layout — two stale-upstream leftovers, and readme.md is the one that will actively mislead you"
links: [langgraph-agent-convention, agent-typescript-parity, env-and-integrations]
absent: workspace:*
cites:
  - package.json:9 :: concurrently
  - package.json:12 :: "dev:agent:py"
  - package.json:14 :: install:agent:ts
  - package.json:17 :: "@copilotkit/react-core"
  - package.json:43 :: "nx"
  - agents/typescript/package.json:13 :: "@copilotkit/sdk-js"
  - agents/typescript/package.json:23 :: "langchain"
  - agents/python/langgraph.json:5 :: graphs
  - agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
  - vercel.json:2 :: nx run
  - next.config.mjs:3 :: standalone
  - agents/python/main.py:41 :: PORT
  - agents/python/uv.lock:1 :: version
  - package.json:13 :: uv sync
  - readme.md:24 :: cd agent-py
  - readme.md:25 :: poetry install
  - readme.md:60 :: cd ./ui
  - readme.md:77 :: remoteEndpoints
  - readme.md:108 :: ./agent-py
---
How the app is run and built.

**Local dev** (`package.json` scripts): `npm run dev` uses `concurrently` (`package.json:9`)
to start two processes — `dev:ui` (`next dev`, `:10`) and `dev:agent` (`:11`) →
`dev:agent:py` (`cd agents/python && uv run main.py`, `package.json:12`). **Neither script
sets its own port**: both ports are injected by the `dev` line itself, which prefixes the two
commands with `PORT=3000` and `PORT=8000` (`package.json:9`) — the agent then reads that as
`int(os.getenv("PORT", "8000"))` (`main.py:41`). So to change a port, edit `:9`, not `:10`
or `:12`; and running `npm run dev:agent:py` alone inherits no `PORT` and falls back to the
`main.py` default. The Python agent is `uv`-managed (`agents/python/uv.lock`, `uv sync` at
`package.json:13`) — **not poetry**; the TS agent is `pnpm`-managed and **not** in the
default `dev` (see
[agent parity](/brain/rules/agent-typescript-parity.md)).

**Agent graph registration**: each backend's `langgraph.json` `graphs` map (`:5`) points both
agent ids at the same `agent.py:graph` / `agent.ts:graph`. The Python graph toggles its
checkpointer on the `LANGGRAPH_FASTAPI` env flag (`agent.py:40`): unset/`false` → no custom
checkpointer (LangGraph API mode); `true` (set by `main.py:12`) → `MemorySaver` (CopilotKit
mode). Know this when the graph behaves differently under `langgraph dev` vs `npm run dev`.

**The de-workspacing is now DONE for both packages — the leftovers are deploy/docs.** This
checkout is a fork of one leaf of the upstream CopilotKit examples monorepo. Both installable
packages have been cut loose from that workspace; two non-package artifacts were not.

Migrated (installs standalone):
- The three CopilotKit deps at the root were **de-workspaced** to real npm ranges — `^1.63.2`
  for `@copilotkit/react-core` (`package.json:17`), `react-ui` (`:18`) and `runtime` (`:19`)
  — and a root `pnpm-lock.yaml` was committed. A plain `pnpm install` here resolves.
- **`agents/typescript` was de-workspaced on 2026-07-28.** `@copilotkit/sdk-js` moved from
  `workspace:*` to `^1.63.2` (`agents/typescript/package.json:13`), so
  `npm run install:agent:ts` (`package.json:14`) now resolves outside the upstream workspace,
  and a `pnpm-lock.yaml` was committed alongside it. `workspace:*` is declared `absent:` on
  this note, so gate B3 re-greps it every run — if any package regains one, this note fails
  until it is re-stated.

  Removing the pin was necessary but **not sufficient** to run the TS agent: `sdk-js@1.63.2`
  requires `@langchain/core`/`@langchain/langgraph` `>=0.4.0`, while the folder was frozen at
  `core@0.3.x` / `langgraph@0.2.44`, so graph import died with `ERR_PACKAGE_PATH_NOT_EXPORTED`.
  The langchain pins were moved to 1.x in the same pass (`@langchain/core:15`,
  `@langchain/langgraph:17`, and `langchain` itself added as a direct dep at `:23` because
  sdk-js peers on it). The graph ran unmodified afterward — no source change was needed. It is
  still not wired into `npm run dev` ([agent parity](/brain/rules/agent-typescript-parity.md)).

Still monorepo-bound (fails outside it):
- **`vercel.json` was not updated**: `installCommand`/`buildCommand` still `cd ../../../`
  and run `npx nx run @copilotkit-examples/research-canvas:build` (`vercel.json:2-3`), and
  `package.json` still carries the `nx` target block (`:43-49`). Those paths only exist
  inside the upstream monorepo, so deploying this repo standalone on Vercel with the
  committed config fails at install. Fix `vercel.json`; don't re-add `workspace:*` to the
  root to make it match.

- **`readme.md` is upstream-monorepo legacy — do not follow it.** It is the single most
  salient "how do I run this" artifact in the repo and it is wrong end to end; the
  `package.json` scripts above are the truth. Every step fails on this checkout:
  - `readme.md:24,31` say `cd agent-py` / `cd agent-js`. Neither directory exists — they are
    `agents/python` and `agents/typescript`.
  - `readme.md:25,52` say `poetry install` / `poetry run demo`. The Python agent is
    `uv`-managed; `poetry` appears **nowhere in code** (declared `absent:` here, so gate B3
    re-greps it every run). Use `npm run install:agent:py` (`package.json:13`).
  - `readme.md:35` puts `.env` inside `./agent-py` or `./agent-js`, and `:64` puts a second
    one inside `./ui` (reached via `cd ./ui`, `readme.md:60`). None of those three paths
    exists — the UI **is** the repo root, and the agent's `.env` belongs in `agents/python`.
  - `readme.md:67` puts `OPENAI_API_KEY` in the UI `.env`. The UI reads no `OPENAI_API_KEY`
    — the only occurrence is commented out (`route.ts:16`). Its env block also **omits
    `NEXT_PUBLIC_COPILOTKIT_API_KEY`**, without which the app 401s on every request; see
    [env-and-integrations](/brain/rules/env-and-integrations.md), which is the authoritative
    env list.
  - `readme.md:77-95` tell you to uncomment `remoteEndpoints` / `langGraphPlatformEndpoint`
    in `route.ts`. **Neither token exists anywhere in code** — both are declared `absent:`
    here so the claim is re-greppable, and the snippet's agent name is the typo
    `research_agentt` (`readme.md:89`). The real mode switch is the `?lgcDeploymentUrl=`
    param ([route convention](/brain/rules/copilotkit-runtime-route-convention.md)).
  - `readme.md:108` points LangGraph Studio at `./agent-py`; load `agents/python` instead.

  Same class as `vercel.json` above and `dockerize.sh` below — a leaf copied out of the
  upstream monorepo without its docs re-pointed. Rewrite it or delete it; don't let a cold
  reader (or agent) mistake its salience for authority.

`next.config.mjs` sets `output: "standalone"` (`:3`) for containerized deploys
(`dockerize.sh`, which itself references `./examples/Dockerfile.ui` — another upstream-relative
path).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [package.json:9](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/package.json#L9) — `concurrently`
[2] [package.json:12](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/package.json#L12) — `"dev:agent:py"`
[3] [package.json:17](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/package.json#L17) — `"@copilotkit/react-core"`
[4] [package.json:43](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/package.json#L43) — `"nx"`
[5] [agents/typescript/package.json:13](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/typescript/package.json#L13) — `workspace:*`
[6] [agents/python/langgraph.json:5](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/langgraph.json#L5) — `graphs`
[7] [agents/python/src/agent.py:40](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L40) — `LANGGRAPH_FASTAPI`
[8] [vercel.json:2](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/vercel.json#L2) — `nx run`
[9] [next.config.mjs:3](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/next.config.mjs#L3) — `standalone`
[10] [agents/python/main.py:41](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/main.py#L41) — `PORT`
[11] [agents/python/uv.lock:1](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/uv.lock#L1) — `version`
[12] [package.json:13](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/package.json#L13) — `uv sync`
[13] [readme.md:24](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/readme.md#L24) — `cd agent-py`
[14] [readme.md:25](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/readme.md#L25) — `poetry install`
[15] [readme.md:60](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/readme.md#L60) — `cd ./ui`
[16] [readme.md:77](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/readme.md#L77) — `remoteEndpoints`
[17] [readme.md:108](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/readme.md#L108) — `./agent-py`

<!-- okf:citations:end -->

