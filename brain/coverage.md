---
schemaVersion: 1
domains: { routing-app-shell: mapped, components: mapped, agent-bridge: mapped, state: mapped, agent-python: mapped, agent-typescript: mapped, external-integrations: mapped, build-tooling: mapped, toolbox: mapped, data-persistence: stubbed, auth: stubbed }
inventory:
  - route-pages :: src/app/**/page.tsx :: 1
  - api-routes :: src/app/api/**/route.ts :: 1
  - feature-components :: src/components/*.tsx :: 6
  - ui-primitives :: src/components/ui/*.tsx :: 6
  - agent-graphs :: agents/**/langgraph.json :: 2
  - py-agent-nodes :: agents/python/src/lib/*.py :: 7
---
# Coverage — founding scan of `@copilotkit-examples/research-canvas`

A single Next.js 15 App Router app (one client-rendered route) bridged via CopilotKit to a
LangGraph research agent that exists in **two** parallel backends (Python — the one `npm run
dev` runs — and an alternate TypeScript port). Breadth mapped before depth; every citation
mechanically verified (see [citation-check](/brain/citation-check.md)).

## Domain status

| Domain | Status | Notes |
| --- | --- | --- |
| routing-app-shell | mapped | `nextjs-app-shell-convention`, `provider-nesting-contract` |
| components | mapped | `design-system-convention` (shadcn new-york + hardcoded brand hex) |
| agent-bridge | mapped | `agent-name-contract`, `delete-resources-hitl-contract`, `copilotkit-runtime-route-convention`, `research-chat-flow`, `model-selection-flow`, `add-agent-tool-howto` |
| state | mapped | `agent-state-shape-contract` (3-place cross-process shape) |
| agent-python | mapped | `langgraph-agent-convention` (primary backend) |
| agent-typescript | mapped | `agent-typescript-parity` (alternate port, not run by default) |
| external-integrations | mapped | `env-and-integrations` (names only, from source) |
| build-tooling | mapped | `build-tooling-convention` (concurrently, workspace:* monorepo, nx/Vercel) |
| toolbox | mapped | `repo-toolbox` (rafa skills + rafinery MCP + permissions) |
| data-persistence | stubbed | No database. Resources/report live only in coagent state; downloaded page content is memoized in an in-process dict `_RESOURCE_CACHE` (`agents/python/src/lib/download.py:17`) that dies with the process. Nothing to map. |
| auth | stubbed | No real authentication. Only the cosmetic `x-api-key` header check against a `NEXT_PUBLIC_` (browser-shipped) key (`src/app/api/copilotkit/route.ts:22`), documented in `copilotkit-runtime-route-convention`. No user/session system exists. |

## Primary vs alternate backend

`agents/python/` is the **actively-run** backend: root `npm run dev` → `dev:agent:py`
(`uv run main.py`, port 8000). `agents/typescript/` is a faithful but **lightly-used** port
(pnpm-installed, requires an explicit route.ts uncomment per readme, uses gpt-4o vs the
Python gpt-4o-mini). Both are mapped; parity drift is called out in `agent-typescript-parity`.

## Acceptance criteria — per-criterion

**A · Coverage** — A1 PASS (single app + both agent packages + toolbox all listed; this is a
monorepo leaf, noted in build-tooling). A2 PASS (every domain has an explicit status). A3
PASS (all 9 mapped domains have ≥1 note). A4 PASS (both stubbed domains state why, cited).

**B · Fidelity** — B1 PASS (`npx @rafinery/cli verify-citations` exits 0; see
[citation-check](/brain/citation-check.md)). B2 PASS (contracts declare `anchor:` — token
anchors `research_agent`, `DeleteResources` with every code occurrence cited; `anchor: none`
with reason for the two composition/shape contracts). B3 N/A (no absence-claims are load-
bearing; stubbed-domain claims are cited to what DOES exist, not asserted absence). B4 PASS
(`agent-state-shape-contract` is a cross-process state-shape contract). B5 PASS
(`provider-nesting-contract` captures provider composition ordering).

**C · Work-time value** — C1 PASS: for the real feature "add a new agent tool"
(`add-agent-tool-howto` → chat.py/agent.py + ResearchCanvas useCopilotAction + state files)
and the real bug "model/report doesn't stream to the canvas" (`research-chat-flow` +
`agent-state-shape-contract` + `agent-name-contract`), the notes route to the exact files
for all four questions without blind search. C2 PASS. C3 PASS (all notes carry type/domain/
≥1 cite; contracts carry `failure:`). C4 PASS (contracts and flows cross-link).

**D · Format** — D1 PASS (`rules/` + `playbooks/` + this file; no graph.json). D2 PASS
(frontmatter per contract §2). D3 PASS (`npx @rafinery/cli compile` exits 0 → `manifest.json`).

## Least-confident areas (for prism)

- **`agent-name-contract` completeness** — 18 cited occurrences of `research_agent`; the
  readme.md typo `research_agentt` is treated as docs/excluded. Confirm the checker's
  docs-exclusion agrees.
- **Toolbox citations into untracked files** — `.claude/**`, `.mcp.json` are not yet
  git-tracked (branch `chore/rafa-init`); citations resolve against the working tree.
- **TS/Python parity drift** — `agent-typescript-parity` claims faithful port + specific
  drifts (gpt-4o vs mini, `route` fn vs `Command`); worth an adversarial recheck.
- **model-selection-flow claim** — that both graph ids resolve to the same graph so no model
  option is truly "dead"; verify this reasoning against CopilotKit's agent-name routing.
