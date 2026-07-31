---
schemaVersion: 1
id: env-and-integrations
type: convention
domain: external-integrations
title: Environment variables and external services this app reads
summary: NEXT_PUBLIC_COPILOTKIT_API_KEY is REQUIRED for the app to function — unset means every POST /api/copilotkit 401s silently; beyond it the route reads LangSmith/deployment keys, each agent reads MODEL + provider keys + TAVILY_API_KEY, OPENAI_API_KEY appears only commented-out in the runtime route while ANTHROPIC_API_KEY has no occurrence in CODE — all provider LLM keys are otherwise read implicitly by the LangChain constructors
links: [copilotkit-runtime-route-convention, langgraph-agent-convention, repo-toolbox, security-posture]
cites:
  - src/app/api/copilotkit/route.ts:25 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
  - src/app/api/copilotkit/route.ts:19 :: LANGSMITH_API_KEY
  - src/app/api/copilotkit/route.ts:102 :: LGC_DEPLOYMENT_URL
  - src/app/api/copilotkit/route.ts:105 :: REMOTE_ACTION_URL
  - agents/python/src/lib/model.py:19 :: MODEL
  - agents/python/src/lib/model.py:42 :: GOOGLE_API_KEY
  - agents/python/src/lib/model.py:26 :: ChatOpenAI
  - agents/python/src/lib/search.py:34 :: TAVILY_API_KEY
  - agents/python/main.py:41 :: PORT
  - agents/typescript/src/model.ts:33 :: GOOGLE_API_KEY
  - agents/typescript/src/search.ts:38 :: TAVILY_API_KEY
  - src/app/api/copilotkit/route.ts:16 :: OPENAI_API_KEY
  - src/app/page.tsx:33 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
  - src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
  - src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
absent: ANTHROPIC_API_KEY
description: "NEXT_PUBLIC_COPILOTKIT_API_KEY is REQUIRED for the app to function — unset means every POST /api/copilotkit 401s silently; beyond it the route reads LangSmith/deployment keys, each agent reads MODEL + provider keys + TAVILY_API_KEY, OPENAI_API_KEY appears only commented-out in the runtime route while ANTHROPIC_API_KEY has no occurrence in CODE — all provider LLM keys are otherwise read implicitly by the LangChain constructors"
tags: [external-integrations]
timestamp: 2026-07-26T22:44:42.840Z
---
Env var **names** and where source reads them (values never inspected; `.env*` never opened).

**Frontend / runtime route** (`src/app/api/copilotkit/route.ts`, `src/app/page.tsx`):
- `NEXT_PUBLIC_COPILOTKIT_API_KEY` — **the one variable the app cannot run without. Set it
  first.** `isAuthorized` is `return Boolean(apiKey) && req.headers.get("x-api-key") === apiKey`
  (`route.ts:26`), so when the var is unset the left conjunct is `false` **unconditionally**
  and *every* `POST /api/copilotkit` returns 401 (`route.ts:87-89`). The failure is silent:
  no startup warning, no server error, no message in the UI — the chat panel simply never
  answers, and nothing else in the app misbehaves, so it reads as "CopilotKit is broken."
  Any value works, because `page.tsx:33` sends back that same env var as the `x-api-key`
  header, so the two sides always match by construction. **There is no `.env.example` in
  this repo** (`coverage.md` inventory row `env-example :: .env.example :: 0` re-counts it
  every run) and `readme.md`'s env template never names this key, so a fresh clone is dead
  by default with no breadcrumb. Treat it as **cosmetic as a security boundary but mandatory
  as a runtime input** — the two claims are both true and are about different things; see
  [the route convention](/brain/rules/copilotkit-runtime-route-convention.md) and
  [the security posture](/brain/playbooks/security-posture.md) for the boundary half.
  `NEXT_PUBLIC_` = **ships to the browser** (`route.ts:25`, `page.tsx:33`); not a secret
  boundary.
- `LANGSMITH_API_KEY` — LangGraph-Cloud auth, only used in `lgcDeploymentUrl` mode
  (`route.ts:19`).
- `LGC_DEPLOYMENT_URL` — optional LangGraph Cloud deployment URL (`route.ts:102`), the
  fallback when no `?lgcDeploymentUrl=` param is supplied.
- `REMOTE_ACTION_URL` — backend agent base URL, default `http://localhost:8000/copilotkit`
  (`route.ts:105`).

**Python agent** (`agents/python/`):
- `MODEL` — overrides `state.model` (`model.py:19`).
- `GOOGLE_API_KEY` — passed explicitly to `ChatGoogleGenerativeAI` (`model.py:42`).
- `TAVILY_API_KEY` — web search client (`search.py:34`).
- `PORT` — uvicorn port, default 8000 (`main.py:41`); `LANGGRAPH_FASTAPI` — set in-process
  to toggle the checkpointer (`main.py:12`, read `agent.py:40`), not a user-facing key.
- **Provider LLM keys**: read by the LangChain SDK constructors themselves (`ChatOpenAI`
  `model.py:26`, `ChatAnthropic` `:30`, `ChatXAI` `:47`), one per model branch — the key is
  required whenever that model is selected, but none is passed by name to the constructor.
  `OPENAI_API_KEY` is the only one literal in **source**, and only **commented-out** (a
  disabled `OpenAIAdapter` wiring, `route.ts:16`). `readme.md`'s env block names four keys
  (OPENAI/TAVILY/XAI/LANGSMITH) — but **do not use it as the setup template**: it is
  upstream-monorepo legacy, its paths and package manager are wrong for this checkout, and
  it omits both `NEXT_PUBLIC_COPILOTKIT_API_KEY` (above — the app is dead without it) and
  `GOOGLE_API_KEY`. It is flagged as a non-exemplar in
  [build-tooling](/brain/rules/build-tooling-convention.md); **this note is the authoritative
  env list.**
  `ANTHROPIC_API_KEY` has **no occurrence anywhere in this repository** — not in code, not
  in docs. It is declared `absent:` in frontmatter so gate B3 re-greps it every run and the
  claim can't silently go stale. Selecting **Anthropic** in the model dropdown therefore
  works only if the key is supplied through the environment, where nothing in this repo
  names it: `ChatAnthropic` reads it implicitly inside the LangChain constructor
  (`model.py:30`), so grepping the codebase for the variable you need returns nothing. The
  same implicit-constructor pattern covers `grok`/`ChatXAI` (`model.py:47`) — consult the
  provider SDK for the exact variable name, because this repo never states it. That is the
  trap this bullet exists to flag.

**TS agent** (`agents/typescript/`): same `MODEL`, `GOOGLE_API_KEY` (`model.ts:33`),
`TAVILY_API_KEY` (`search.ts:38`), plus the same implicit provider keys via its `@langchain/*`
constructors.

**External services**: Tavily (search), the four LLM providers (OpenAI/Anthropic/Google/xAI),
optional LangGraph Cloud + LangSmith. The rafinery MCP key (`RAFA_MCP_KEY`) is tooling, not
app — see [the repo toolbox](/brain/rules/repo-toolbox.md). Which of these are
server-only vs browser-visible is mapped in
[the security posture](/brain/playbooks/security-posture.md).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/api/copilotkit/route.ts:25](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L25) — `NEXT_PUBLIC_COPILOTKIT_API_KEY`
[2] [src/app/api/copilotkit/route.ts:19](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L19) — `LANGSMITH_API_KEY`
[3] [src/app/api/copilotkit/route.ts:102](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L102) — `LGC_DEPLOYMENT_URL`
[4] [src/app/api/copilotkit/route.ts:105](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L105) — `REMOTE_ACTION_URL`
[5] [agents/python/src/lib/model.py:19](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L19) — `MODEL`
[6] [agents/python/src/lib/model.py:42](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L42) — `GOOGLE_API_KEY`
[7] [agents/python/src/lib/model.py:26](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L26) — `ChatOpenAI`
[8] [agents/python/src/lib/search.py:34](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/search.py#L34) — `TAVILY_API_KEY`
[9] [agents/python/main.py:41](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/main.py#L41) — `PORT`
[10] [agents/typescript/src/model.ts:33](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/model.ts#L33) — `GOOGLE_API_KEY`
[11] [agents/typescript/src/search.ts:38](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/search.ts#L38) — `TAVILY_API_KEY`
[12] [src/app/api/copilotkit/route.ts:16](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L16) — `OPENAI_API_KEY`
[13] [src/app/page.tsx:33](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/page.tsx#L33) — `NEXT_PUBLIC_COPILOTKIT_API_KEY`
[14] [src/app/api/copilotkit/route.ts:26](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L26) — `Boolean(apiKey)`
[15] [src/app/api/copilotkit/route.ts:87-89](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L87-L89) — `Unauthorized`

<!-- okf:citations:end -->
