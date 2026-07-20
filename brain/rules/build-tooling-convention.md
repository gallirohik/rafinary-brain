---
schemaVersion: 1
id: build-tooling-convention
type: convention
domain: build-tooling
title: Dev/build wiring — concurrently runs UI + Python agent; workspace:* means it lives in a monorepo
summary: npm run dev launches Next.js (3000) and the Python agent (8000) together; CopilotKit deps are workspace:* so this app is really a package inside the CopilotKit monorepo, and Vercel builds it via nx from the repo root
links: [langgraph-agent-convention, agent-typescript-parity]
cites:
  - package.json:9 :: concurrently
  - package.json:12 :: "dev:agent:py"
  - package.json:17 :: "workspace:*"
  - agents/python/langgraph.json:5 :: graphs
  - agents/python/src/agent.py:37 :: LANGGRAPH_FASTAPI
  - vercel.json:2 :: nx run
  - next.config.mjs:3 :: standalone
---
How the app is run and built.

**Local dev** (`package.json` scripts): `npm run dev` uses `concurrently` (`package.json:6`)
to start two processes — `dev:ui` (`next dev`, port 3000) and `dev:agent` → `dev:agent:py`
(`cd agents/python && uv run main.py`, port 8000) (`package.json:9`). The Python agent is
`uv`-managed; the TS agent is `pnpm`-managed and **not** in the default `dev` (see
[agent parity](/brain/rules/agent-typescript-parity.md)).

**Agent graph registration**: each backend's `langgraph.json` `graphs` map (`:5`) points both
agent ids at the same `agent.py:graph` / `agent.ts:graph`. The Python graph toggles its
checkpointer on the `LANGGRAPH_FASTAPI` env flag (`agent.py:37`): unset/`false` → no custom
checkpointer (LangGraph API mode); `true` (set by `main.py`) → `MemorySaver` (CopilotKit
mode). Know this when the graph behaves differently under `langgraph dev` vs `npm run dev`.

**This is a monorepo package, not a standalone repo.** The CopilotKit deps are
`"workspace:*"` (`package.json:19+`: `@copilotkit/react-core`, `react-ui`, `runtime`) — they
resolve from a parent pnpm workspace, so `npm install` here in isolation will not satisfy
them. `vercel.json` confirms it: build/install `cd ../../../` and run
`nx run @copilotkit-examples/research-canvas:build` (`vercel.json:2`). `next.config.mjs`
sets `output: "standalone"` (`:3`) for containerized deploys (`dockerize.sh`). Treat the
checked-out folder as one leaf of the upstream CopilotKit examples monorepo.
