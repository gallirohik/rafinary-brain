---
schemaVersion: 1
domains: { routing-app-shell: mapped, components: mapped, state: mapped, agent-bridge: mapped, agent-python: mapped, agent-typescript: mapped, external-integrations: mapped, security: mapped, build-tooling: mapped, toolbox: mapped, auth: empty, data-persistence: empty, testing: empty }
inventory:
  - route-pages :: src/app/**/page.tsx :: 1
  - api-routes :: src/app/api/**/route.ts :: 1
  - middleware :: **/middleware.ts :: 0
  - env-example :: .env.example :: 0
  - feature-components :: src/components/*.tsx :: 7
  - ui-primitives :: src/components/ui/*.tsx :: 6
  - agent-graph-manifests :: agents/*/langgraph.json :: 2
  - python-agent-nodes :: agents/python/src/lib/*.py :: 8
  - rafa-skills :: .claude/skills/*/SKILL.md :: 13
  - agent-skills :: .agents/skills/*/SKILL.md :: 7
---

# Coverage — research-canvas

**This pass was a REFRESH, not a founding scan.** The org brain (15 notes / 9 improvements)
was restored from `main` and re-verified note-by-note against current source. All 15
existing notes were re-verified and drift-corrected in place; 1 note was added
(`security-posture`, the domain the older SOP revision did not require), for **16 notes on
disk** — the count `rafa compile` re-derives and the count used everywhere below. No note
was duplicated or replaced by a differently-named parallel file.

`coverage.md` itself is new — the revision of the scan SOP that produced this brain did not
require it, so the manifest carried no `domains`/`inventory` tracking. This file supplies
both for the full current state.

## Repo shape

Single application, no workspace config of its own (`pnpm-workspace.yaml` / `turbo.json`
are absent — this checkout is a de-workspaced fork of one leaf of the upstream CopilotKit
examples monorepo, see [build-tooling-convention](/brain/rules/build-tooling-convention.md)).
Three code units, all covered:

| unit | what | notes |
| --- | --- | --- |
| `src/` | Next.js 15 App Router UI, one route | routing-app-shell, components, state |
| `agents/python/` | the **live** LangGraph backend (`npm run dev` starts it) | agent-python |
| `agents/typescript/` | an accepted-lagging port, not started by default | agent-typescript |

## Domain status

| domain | status | notes |
| --- | --- | --- |
| routing-app-shell | mapped | `nextjs-app-shell-convention`, `provider-nesting-contract` |
| components | mapped | `design-system-convention` |
| state | mapped | `agent-state-shape-contract` |
| agent-bridge | mapped | `copilotkit-runtime-route-convention`, `agent-name-contract`, `delete-resources-hitl-contract`, `research-chat-flow`, `model-selection-flow`, `add-agent-tool-howto` |
| agent-python | mapped | `langgraph-agent-convention` |
| agent-typescript | mapped | `agent-typescript-parity` |
| external-integrations | mapped | `env-and-integrations` |
| security | mapped | `security-posture` (**new this pass**) |
| build-tooling | mapped | `build-tooling-convention` |
| toolbox | mapped | `repo-toolbox` |
| auth | **empty** | There is no authentication system to map — and the emptiness is now **gate-enforced, not grep-proven-once**: `security-posture` declares `getServerSession`, `next-auth`, `@clerk`, `useSession`, `cookies(`, `next/headers`, `Depends` and `add_middleware` as `absent:` tokens, so gate B3 re-greps all eight against code every run and this row fails loudly the day identity lands. The only gate today is the `x-api-key` in the runtime route — cosmetic as a boundary but **required to be set or every request 401s** (`env-and-integrations`). The absence and its blast radius are documented in `security-posture` rather than given a stub note of their own. |
| data-persistence | **empty** | Nothing is persisted. Coagent state lives in LangGraph's in-process `MemorySaver`; downloaded pages live in the module-level `_RESOURCE_CACHE`; both die with the process. No ORM/client of any kind in code — the only hit for a database token repo-wide is an unused `langgraph-checkpoint-sqlite`/`aiosqlite` dependency declared in `agents/python/pyproject.toml:24-25` and never imported. Covered as a property in `langgraph-agent-convention` + `build-tooling-convention`; a note would describe nothing. |
| testing | **empty** | No test harness exists. `git ls-files` matches no `*.test.*`, `*.spec.*`, `__tests__/`, `test_*.py` or `conftest.py`; the root `package.json` has no test script and `pyproject.toml` no pytest config. The only `test` script in the repo is the npm-init placeholder in `agents/typescript/package.json:10` (`echo "Error: no test specified" && exit 1`) — not a harness. (The `describe(`/`.test(` grep hits in `search.ts` and `route.ts` are zod `.describe()` and `RegExp.test()`.) Nothing to map; flagged so the gap is explicit rather than silent. |

No domain is silently omitted. Every `mapped` domain has ≥1 note; every `empty` domain
states why above.

## Acceptance criteria

**A · Coverage (breadth)**
- **A1 PASS** — every code unit in the repo (`src/`, `agents/python/`, `agents/typescript/`)
  is listed and covered. There is no workspace config enumerating packages; the units above
  are the whole tree.
- **A2 PASS** — 13 domains, each with an explicit status (10 `mapped`, 3 `empty`).
- **A3 PASS** — every `mapped` domain has ≥1 note (table above).
- **A4 PASS** — all three `empty` domains state why, each backed by a grep, not an
  impression.

**B · Fidelity**
- **B1 PASS** — `npx @rafinery/cli verify-citations` exits 0; every `file:line :: token`
  resolved mechanically. The full per-cite table is pasted below in
  [Appendix B1](#appendix-b1--full-citation-table). The same table is emitted, timestamped,
  to [`citation-check.md`](/brain/citation-check.md) / `citation-check.json` on every run —
  that generated pair is the authority if the paste below ever disagrees with it.
- **B2 PASS** — 4 contracts, all declaring `anchor:`. Token anchors `research_agent` and
  `DeleteResources` have every code occurrence cited (the checker re-greps and would fail on
  an omitted site — it did fail this pass, on 11 sites that moved, until they were
  re-cited). `agent-state-shape-contract` and `provider-nesting-contract` declare
  `anchor: none` with a reason (cross-process field set; provider nesting order).
- **B3 PASS** — **thirteen** absence tokens, each grep-proven and declared so the checker
  re-greps it forever:
  - `ANTHROPIC_API_KEY` (env-and-integrations).
  - `add_middleware`, `use server`, and — added in round 3 after prism found the
    auth-identity absence stated in prose but ungated — `getServerSession`, `next-auth`,
    `@clerk`, `useSession`, `cookies(`, `next/headers`, `Depends` (all security-posture).
    That set is the mechanical form of "there is no identity in this app": the provider,
    the hook, the server-side session helper, the request-context import and its cookie
    accessor, plus both FastAPI ways to attach a guard. The `auth: empty` row above now
    rests on those greps rather than on a one-time check.
  - `remoteEndpoints`, `langGraphPlatformEndpoint`, `poetry` (build-tooling-convention) —
    the three tokens `readme.md` instructs a reader to use that exist nowhere in code, so
    the non-exemplar flag on that file is itself gate-backed.

  (`use server` was added in round 2; prism found the note *asserting* it was declared while
  it was not, the exact durability failure the declaration exists to prevent.)
  Two further absence-shaped claims in security-posture are
  carried by the **inventory** lane instead, because they are about files rather than
  tokens: `middleware :: **/middleware.ts :: 0` (no Next.js middleware) and
  `api-routes :: src/app/api/**/route.ts :: 1` (exactly one route handler) — `git ls-files`
  re-counts both every run. Round 3 added a third file-shaped one,
  `env-example :: .env.example :: 0`, so env-and-integrations' "a fresh clone has no env
  template to copy" claim is re-counted rather than remembered. A token `workspace:*` was drafted and the checker
  **refuted it** —
  `agents/typescript/package.json:13` still carries one. The note was corrected to state the
  narrower, true claim (root de-workspaced, TS agent not) and `workspace:*` was promoted from
  `absent:` to that note's `anchor:`, so completeness is now enforced instead of absence. No
  un-declared absence-shaped claim remains; checker WARN lane is empty.
- **B4 PASS** — `agent-state-shape-contract` captures the cross-process field set (frontend
  TS type ↔ Python TypedDict ↔ TS annotation), including the deliberate `citations` lag.
- **B5 PASS** — `provider-nesting-contract` captures the composition/ordering invariant
  (ModelSelectorProvider → context consumers → CopilotKit → coagent hooks).

**C · Work-time value**
- **C1 PASS** — probed with one real feature ("add a tool that streams a new state field")
  and one real bug ("the Fact Check panel never renders"). Feature: `add-agent-tool-howto`
  → `langgraph-agent-convention` → `agent-state-shape-contract` names every file to touch,
  including the `emit_intermediate_state` mapping and the frontend action-name coupling.
  Bug: `research-chat-flow` step 4/5 → `agent-state-shape-contract` (citations is
  Python-only) → `agent-typescript-parity` (the TS port has no `fact_check_node`) lands on
  the cause without a search. Round 3 closed prism's third probe — "fresh clone, nothing
  works": `env-and-integrations` now leads with `NEXT_PUBLIC_COPILOTKIT_API_KEY` as a
  **required runtime input** (unset ⇒ `Boolean(apiKey)` false ⇒ every request 401s silently,
  `route.ts:26` / `:87-89`), cross-stated in `copilotkit-runtime-route-convention` and
  `security-posture` so all three places that call the gate "cosmetic" now separate
  *weak boundary* from *optional to set*.
- **C2 PASS** — every note answers ≥1 of the four work-time questions; none is a code
  description.
- **C3 PASS** — all 16 notes carry `type`, `domain`, ≥1 `cite`; all 4 contracts carry
  `failure:` — `agent-name-contract` `silent`, `delete-resources-hitl-contract` `silent`,
  `provider-nesting-contract` `loud`, `agent-state-shape-contract` `silent` (added this
  round: prism found it missing — the field set mismatches across the process boundary and
  simply doesn't render, so `silent` is the honest value). Note `contract.md` §2 marks
  `failure:` *optional*, so `rafa compile` cannot catch this class; the check is human until
  the SOP and the compiler are reconciled.
- **C4 PASS** — contracts and flows cross-link bundle-relative; blast radius is traversable
  (e.g. `agent-name-contract` ↔ `research-chat-flow` ↔ `copilotkit-runtime-route-convention`
  ↔ `security-posture`).

**D · Format & contract**
- **D1 PASS** — output is `brain/rules/` (12) + `brain/playbooks/` (4) + this
  `coverage.md`. No `graph.json`.
- **D2 PASS** — frontmatter valid per contract §2 on all 16 note files.
- **D3 PASS** — `npx @rafinery/cli compile` exits 0 and writes `.rafa/manifest.json`.

## Drift corrected this pass

Code moved under the brain since these notes were written (chiefly commit `6c5fac0`
"resolved improvements", which rewrote `route.ts` and `layout.tsx` and de-workspaced
`package.json`, and the `verifiable-report` epic, which added the fact-check node). 29
citations had gone stale and 11 anchor sites had moved out from under their contracts;
all were re-resolved against current source. Four claims were not merely off-by-N but
**wrong** and were rewritten: the `workspace:*` monorepo claim, the "all feature components
are `use client`" claim, the TS port's "same five nodes / same tools" parity claim, and the
runtime route's hostname-string SSRF description (the guard now resolves DNS). Details in
the refresh summary.

**Round 3 (prism ITERATE, score 92).** Three majors and three minors closed, no new notes:
the `NEXT_PUBLIC_COPILOTKIT_API_KEY` 401-lockout consequence added to
`env-and-integrations` / `copilotkit-runtime-route-convention` / `security-posture` (M1);
`readme.md` flagged as an upstream-legacy **non-exemplar** in `build-tooling-convention`
alongside `vercel.json` and `dockerize.sh`, with the approving `readme.md:38-41,67` citation
in `env-and-integrations` demoted to "names four keys, but do not use it as the template"
(M2); seven auth-identity `absent:` tokens declared on `security-posture` (M3); three body
links retitled to match their targets, `model-selection-flow`'s debug guidance re-ordered to
put the `MODEL` env override first, and the port-8000 attribution moved from
`package.json:12` to `:9` where `PORT=8000` is actually set (m1–m3).

## Known thinness (honest)

- `agents/typescript/` is mapped by one convention note, deliberately — it is not the live
  backend and mapping it to Python's depth would be documenting code nobody runs. The note
  says exactly where it lags.
- The shadcn `ui/` primitives are covered as a tier/convention, not per-component. They are
  generated files; per-file notes would rot and add nothing over the generator.

## Non-exemplars (salient but wrong — do not copy)

Three committed artifacts read as authority and are not. All three are upstream-monorepo
leftovers, all three are called out in
[build-tooling-convention](/brain/rules/build-tooling-convention.md):

| artifact | why it misleads |
| --- | --- |
| `readme.md` | wrong end to end for this checkout — `cd agent-py` / `poetry install` (it is `agents/python`, `uv`-managed), `.env` in nonexistent `./ui`, and an instruction to uncomment `remoteEndpoints`/`langGraphPlatformEndpoint` tokens that exist nowhere in code. Its env template also omits the one key the app cannot run without. |
| `vercel.json` | `installCommand`/`buildCommand` still `cd ../../../` into the upstream workspace and run `nx`; deploying standalone fails at install. |
| `dockerize.sh` | references `./examples/Dockerfile.ui`, an upstream-relative path. |

## Appendix B1 — full citation table

Pasted verbatim from the checker's own output (`npx @rafinery/cli verify-citations`, exit 0)
so criterion B1 is satisfied in this file rather than by reference. Regenerate with that
command; the timestamped copy lives in `citation-check.md` / `citation-check.json`.

## Resolution (B1): 183/183 ✓
✓ src/lib/model-selector-provider.tsx:42 :: research_agent
✓ src/lib/model-selector-provider.tsx:44 :: research_agent
✓ src/app/api/copilotkit/route.ts:109 :: research_agent
✓ src/app/api/copilotkit/route.ts:110 :: research_agent
✓ src/app/api/copilotkit/route.ts:112 :: research_agent
✓ src/app/api/copilotkit/route.ts:113 :: research_agent
✓ src/app/api/copilotkit/route.ts:121 :: research_agent
✓ src/app/api/copilotkit/route.ts:124 :: research_agent
✓ src/app/api/copilotkit/route.ts:126 :: research_agent
✓ src/app/api/copilotkit/route.ts:129 :: research_agent
✓ agents/python/langgraph.json:6 :: research_agent
✓ agents/python/langgraph.json:7 :: research_agent
✓ agents/python/main.py:20 :: research_agent
✓ agents/python/main.py:22 :: research_agent
✓ agents/python/main.py:27 :: research_agent
✓ agents/python/main.py:29 :: research_agent
✓ agents/typescript/langgraph.json:6 :: research_agent
✓ agents/typescript/langgraph.json:7 :: research_agent
✓ src/lib/types.ts:19 :: AgentState
✓ src/lib/types.ts:21 :: research_question
✓ src/lib/types.ts:22 :: report
✓ src/lib/types.ts:23 :: resources
✓ src/lib/types.ts:24 :: logs
✓ src/lib/types.ts:25 :: citations
✓ agents/python/src/lib/state.py:41 :: AgentState
✓ agents/python/src/lib/state.py:48 :: research_question
✓ agents/python/src/lib/state.py:51 :: logs
✓ agents/python/src/lib/state.py:52 :: citations
✓ agents/typescript/src/state.ts:19 :: AgentStateAnnotation
✓ agents/typescript/src/state.ts:21 :: research_question
✓ agents/typescript/src/state.ts:24 :: logs
✓ agents/typescript/src/agent.ts:15 :: StateGraph
✓ agents/typescript/src/agent.ts:23 :: addConditionalEdges
✓ agents/typescript/src/agent.ts:34 :: interruptAfter
✓ agents/typescript/src/model.ts:20 :: openai
✓ agents/typescript/src/model.ts:40 :: throw new Error
✓ agents/typescript/src/state.ts:19 :: AgentStateAnnotation
✓ package.json:9 :: concurrently
✓ package.json:12 :: "dev:agent:py"
✓ package.json:17 :: "@copilotkit/react-core"
✓ package.json:43 :: "nx"
✓ agents/typescript/package.json:13 :: workspace:*
✓ agents/python/langgraph.json:5 :: graphs
✓ agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
✓ vercel.json:2 :: nx run
✓ next.config.mjs:3 :: standalone
✓ agents/python/main.py:41 :: PORT
✓ agents/python/uv.lock:1 :: version
✓ package.json:13 :: uv sync
✓ readme.md:24 :: cd agent-py
✓ readme.md:25 :: poetry install
✓ readme.md:60 :: cd ./ui
✓ readme.md:77 :: remoteEndpoints
✓ readme.md:108 :: ./agent-py
✓ src/app/api/copilotkit/route.ts:18 :: EmptyAdapter
✓ src/app/api/copilotkit/route.ts:86 :: POST
✓ src/app/api/copilotkit/route.ts:105 :: REMOTE_ACTION_URL
✓ src/app/api/copilotkit/route.ts:107 :: CopilotRuntime
✓ src/app/api/copilotkit/route.ts:135 :: copilotRuntimeNextJSAppRouterEndpoint
✓ src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
✓ src/app/api/copilotkit/route.ts:74 :: dns.lookup
✓ src/app/page.tsx:28 :: CopilotKit
✓ src/app/page.tsx:29 :: runtimeUrl
✓ src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
✓ src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
✓ src/components/ResearchCanvas.tsx:38 :: DeleteResources
✓ agents/python/src/lib/chat.py:32 :: DeleteResources
✓ agents/python/src/lib/chat.py:87 :: DeleteResources
✓ agents/python/src/lib/chat.py:156 :: DeleteResources
✓ agents/typescript/src/agent.ts:55 :: DeleteResources
✓ agents/typescript/src/chat.ts:37 :: DeleteResources
✓ agents/typescript/src/chat.ts:38 :: DeleteResources
✓ agents/typescript/src/chat.ts:57 :: DeleteResources
✓ agents/typescript/src/chat.ts:82 :: DeleteResources
✓ components.json:3 :: new-york
✓ src/components/ui/button.tsx:7 :: buttonVariants
✓ src/lib/utils.ts:4 :: cn
✓ tailwind.config.ts:13 :: hsl(var(--background))
✓ src/app/globals.css:41 :: --radius
✓ src/components/AddResourceDialog.tsx:35 :: #6766FC
✓ src/app/Main.tsx:21 :: #0E103D
✓ src/components/ResearchCanvas.tsx:1 :: "use client"
✓ src/app/api/copilotkit/route.ts:25 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
✓ src/app/api/copilotkit/route.ts:19 :: LANGSMITH_API_KEY
✓ src/app/api/copilotkit/route.ts:102 :: LGC_DEPLOYMENT_URL
✓ src/app/api/copilotkit/route.ts:105 :: REMOTE_ACTION_URL
✓ agents/python/src/lib/model.py:19 :: MODEL
✓ agents/python/src/lib/model.py:42 :: GOOGLE_API_KEY
✓ agents/python/src/lib/model.py:26 :: ChatOpenAI
✓ agents/python/src/lib/search.py:34 :: TAVILY_API_KEY
✓ agents/python/main.py:41 :: PORT
✓ agents/typescript/src/model.ts:33 :: GOOGLE_API_KEY
✓ agents/typescript/src/search.ts:38 :: TAVILY_API_KEY
✓ src/app/api/copilotkit/route.ts:16 :: OPENAI_API_KEY
✓ src/app/page.tsx:33 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
✓ src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
✓ src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
✓ agents/python/src/agent.py:18 :: StateGraph
✓ agents/python/src/agent.py:27 :: set_entry_point
✓ agents/python/src/agent.py:36 :: interrupt_after
✓ agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
✓ agents/python/src/agent.py:13 :: fact_check_node
✓ agents/python/src/agent.py:24 :: fact_check_node
✓ agents/python/src/agent.py:31 :: fact_check_node
✓ agents/python/src/lib/chat.py:37 :: FactCheckReport
✓ agents/python/src/lib/chat.py:88 :: FactCheckReport
✓ agents/python/src/lib/chat.py:160 :: FactCheckReport
✓ agents/python/src/lib/fact_check.py:44 :: ExtractClaimChecks
✓ agents/python/src/lib/model.py:23 :: openai
✓ agents/python/src/lib/model.py:49 :: raise ValueError
✓ agents/python/src/lib/search.py:35 :: TavilyClient
✓ agents/python/src/lib/download.py:30 :: _is_safe_url
✓ agents/python/src/lib/search.py:67 :: if not ai_message.tool_calls
✓ agents/python/src/lib/search.py:139 :: if not ai_message_response.tool_calls
✓ agents/python/src/lib/delete.py:26 :: if ai_message.tool_calls
✓ agents/python/src/lib/fact_check.py:65 :: ai_message.tool_calls[0]["id"]
✓ agents/python/src/lib/fact_check.py:114 :: ai_message.tool_calls[0]["id"]
✓ agents/python/src/lib/fact_check.py:126 :: if not ai_message_response.tool_calls
✓ agents/python/src/lib/fact_check.py:129 :: ai_message.tool_calls[0]["id"]
✓ agents/python/src/lib/fact_check.py:141 :: ai_message.tool_calls[0]["id"]
✓ agents/python/src/lib/download.py:24 :: _RESOURCE_CACHE.get(url, "")
✓ agents/python/src/lib/download.py:66 :: _RESOURCE_CACHE[url] = "ERROR"
✓ agents/python/src/lib/download.py:81 :: _RESOURCE_CACHE[url] = "ERROR"
✓ agents/python/src/lib/download.py:97 :: if not get_resource(resource["url"])
✓ agents/python/src/lib/chat.py:72 :: if content == "ERROR"
✓ agents/python/src/lib/fact_check.py:81 :: if content in ("", "ERROR")
✓ src/app/layout.tsx:22 :: RootLayout
✓ src/app/layout.tsx:6 :: localFont
✓ src/app/page.tsx:1 :: "use client"
✓ src/app/Main.tsx:8 :: Main
✓ src/app/globals.css:15 :: @layer base
✓ src/app/page.tsx:13 :: ModelSelectorProvider
✓ src/app/page.tsx:21 :: useModelSelectorContext
✓ src/app/page.tsx:28 :: CopilotKit
✓ src/app/page.tsx:36 :: Main
✓ src/lib/model-selector-provider.tsx:66 :: throw new Error
✓ .claude/skills/rafa-scan/SKILL.md:2 :: rafa-scan
✓ .claude/commands/rafa.md:2 :: version
✓ .mcp.json:3 :: rafinery
✓ .mcp.json:7 :: RAFA_MCP_KEY
✓ .claude/settings.json:5 :: @rafinery/cli
✓ .claude/settings.json:4 :: Read(.rafa
✓ .claude/settings.json:10 :: SessionStart
✓ .agents/skills/tdd/SKILL.md:2 :: tdd
✓ .agents/skills/vercel-composition-patterns/SKILL.md:2 :: vercel-composition-patterns
✓ rafa.json:11 :: skills
✓ agents/python/src/lib/chat.py:16 :: @tool
✓ agents/python/src/lib/chat.py:82 :: bind_tools
✓ agents/python/src/lib/chat.py:156 :: DeleteResources
✓ agents/python/src/lib/chat.py:52 :: state_key
✓ src/components/ResearchCanvas.tsx:37 :: useCopilotAction
✓ src/components/ModelSelector.tsx:23 :: openai
✓ src/lib/model-selector-provider.tsx:27 :: coAgentsModel
✓ src/lib/model-selector-provider.tsx:42 :: research_agent
✓ src/app/Main.tsx:12 :: model
✓ agents/python/src/lib/model.py:18 :: state.get
✓ agents/python/src/lib/model.py:19 :: MODEL
✓ agents/python/main.py:11 :: load_dotenv
✓ src/app/Main.tsx:45 :: CopilotChat
✓ src/app/api/copilotkit/route.ts:141 :: handleRequest
✓ agents/python/src/lib/chat.py:82 :: bind_tools
✓ agents/python/src/lib/chat.py:48 :: copilotkit_customize_config
✓ agents/python/src/lib/chat.py:160 :: FactCheckReport
✓ agents/python/src/lib/fact_check.py:137 :: citations
✓ agents/python/src/lib/search.py:81 :: copilotkit_emit_state
✓ src/components/ResearchCanvas.tsx:27 :: useCoAgentStateRender
✓ src/components/ResearchCanvas.tsx:193 :: report
✓ src/components/ResearchCanvas.tsx:202 :: citations
✓ src/app/api/copilotkit/route.ts:86 :: export const POST
✓ src/app/api/copilotkit/route.ts:24 :: isAuthorized
✓ src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
✓ src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
✓ src/app/api/copilotkit/route.ts:22 :: visible to anyone via devtools
✓ src/app/page.tsx:33 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
✓ src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
✓ src/app/api/copilotkit/route.ts:19 :: langsmithApiKey
✓ agents/python/main.py:17 :: add_langgraph_fastapi_endpoint
✓ agents/python/main.py:44 :: 0.0.0.0
✓ agents/python/main.py:11 :: load_dotenv
✓ agents/python/src/lib/download.py:30 :: _is_safe_url
✓ agents/python/src/lib/download.py:41 :: getaddrinfo
✓ agents/python/langgraph.json:9 :: .env
✓ .gitignore:38 :: .env

## Completeness (B2): 28/28 ✓  (3 anchors)
✓ anchor 'research_agent' → agents/python/langgraph.json:6
✓ anchor 'research_agent' → agents/python/langgraph.json:7
✓ anchor 'research_agent' → agents/python/main.py:20
✓ anchor 'research_agent' → agents/python/main.py:22
✓ anchor 'research_agent' → agents/python/main.py:27
✓ anchor 'research_agent' → agents/python/main.py:29
✓ anchor 'research_agent' → agents/typescript/langgraph.json:6
✓ anchor 'research_agent' → agents/typescript/langgraph.json:7
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:109
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:110
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:112
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:113
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:121
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:124
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:126
✓ anchor 'research_agent' → src/app/api/copilotkit/route.ts:129
✓ anchor 'research_agent' → src/lib/model-selector-provider.tsx:42
✓ anchor 'research_agent' → src/lib/model-selector-provider.tsx:44
✓ anchor 'workspace:*' → agents/typescript/package.json:13
✓ anchor 'DeleteResources' → agents/python/src/lib/chat.py:32
✓ anchor 'DeleteResources' → agents/python/src/lib/chat.py:87
✓ anchor 'DeleteResources' → agents/python/src/lib/chat.py:156
✓ anchor 'DeleteResources' → agents/typescript/src/agent.ts:55
✓ anchor 'DeleteResources' → agents/typescript/src/chat.ts:37
✓ anchor 'DeleteResources' → agents/typescript/src/chat.ts:38
✓ anchor 'DeleteResources' → agents/typescript/src/chat.ts:57
✓ anchor 'DeleteResources' → agents/typescript/src/chat.ts:82
✓ anchor 'DeleteResources' → src/components/ResearchCanvas.tsx:38

## Policy (contract → anchor declared): 4/4 ✓
✓ rules/agent-name-contract.md
✓ rules/agent-state-shape-contract.md
✓ rules/delete-resources-hitl-contract.md
✓ rules/provider-nesting-contract.md

## Absence (B3, declared `absent:` re-grepped): 13/13 ✓
✓ absent 'remoteEndpoints' → (nowhere — as claimed)
✓ absent 'langGraphPlatformEndpoint' → (nowhere — as claimed)
✓ absent 'poetry' → (nowhere — as claimed)
✓ absent 'ANTHROPIC_API_KEY' → (nowhere — as claimed)
✓ absent 'add_middleware' → (nowhere — as claimed)
✓ absent 'use server' → (nowhere — as claimed)
✓ absent 'getServerSession' → (nowhere — as claimed)
✓ absent 'next-auth' → (nowhere — as claimed)
✓ absent '@clerk' → (nowhere — as claimed)
✓ absent 'useSession' → (nowhere — as claimed)
✓ absent 'cookies(' → (nowhere — as claimed)
✓ absent 'next/headers' → (nowhere — as claimed)
✓ absent 'Depends' → (nowhere — as claimed)

## Inventory (coverage declared vs `git ls-files`): 10/10 ✓
✓ route-pages 'src/app/**/page.tsx' declared 1 · found 1
✓ api-routes 'src/app/api/**/route.ts' declared 1 · found 1
✓ middleware '**/middleware.ts' declared 0 · found 0
✓ env-example '.env.example' declared 0 · found 0
✓ feature-components 'src/components/*.tsx' declared 7 · found 7
✓ ui-primitives 'src/components/ui/*.tsx' declared 6 · found 6
✓ agent-graph-manifests 'agents/*/langgraph.json' declared 2 · found 2
✓ python-agent-nodes 'agents/python/src/lib/*.py' declared 8 · found 8
✓ rafa-skills '.claude/skills/*/SKILL.md' declared 13 · found 13
✓ agent-skills '.agents/skills/*/SKILL.md' declared 7 · found 7

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 0

**All pass.**
