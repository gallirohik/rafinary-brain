---
schemaVersion: 1
id: env-and-integrations
type: convention
domain: external-integrations
title: Environment variables and external services this app reads
summary: Frontend route reads CopilotKit/LangSmith/deployment keys; each agent reads MODEL + provider keys + TAVILY_API_KEY; OPENAI_API_KEY appears commented-out in the runtime route (and with XAI in the readme env template) while ANTHROPIC_API_KEY has no repo occurrence — all provider LLM keys are otherwise read implicitly by the LangChain constructors
links: [copilotkit-runtime-route-convention, langgraph-agent-convention, repo-toolbox]
cites:
  - src/app/api/copilotkit/route.ts:23 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
  - src/app/api/copilotkit/route.ts:17 :: LANGSMITH_API_KEY
  - src/app/api/copilotkit/route.ts:66 :: LGC_DEPLOYMENT_URL
  - src/app/api/copilotkit/route.ts:69 :: REMOTE_ACTION_URL
  - agents/python/src/lib/model.py:19 :: MODEL
  - agents/python/src/lib/model.py:42 :: GOOGLE_API_KEY
  - agents/python/src/lib/model.py:26 :: ChatOpenAI
  - agents/python/src/lib/search.py:34 :: TAVILY_API_KEY
  - agents/python/main.py:41 :: PORT
  - agents/typescript/src/model.ts:33 :: GOOGLE_API_KEY
  - src/app/api/copilotkit/route.ts:14 :: OPENAI_API_KEY
absent: ANTHROPIC_API_KEY
---
Env var **names** and where source reads them (values never inspected; `.env*` never opened).

**Frontend / runtime route** (`src/app/api/copilotkit/route.ts`, `src/app/page.tsx`):
- `NEXT_PUBLIC_COPILOTKIT_API_KEY` — the cosmetic `x-api-key` gate (`route.ts:23`,
  `page.tsx:33`). `NEXT_PUBLIC_` = **ships to the browser**; not a secret boundary.
- `LANGSMITH_API_KEY` — LangGraph-Cloud auth, only used in `lgcDeploymentUrl` mode
  (`route.ts:17`).
- `LGC_DEPLOYMENT_URL` — optional LangGraph Cloud deployment URL (`route.ts:66`).
- `REMOTE_ACTION_URL` — backend agent base URL, default `http://localhost:8000/copilotkit`
  (`route.ts:69`).

**Python agent** (`agents/python/`):
- `MODEL` — overrides `state.model` (`model.py:19`).
- `GOOGLE_API_KEY` — passed explicitly to `ChatGoogleGenerativeAI` (`model.py:42`).
- `TAVILY_API_KEY` — web search client (`search.py:34`).
- `PORT` — uvicorn port, default 8000 (`main.py:41`); `LANGGRAPH_FASTAPI` — set in-process
  to toggle the checkpointer (`main.py:12`, read `agent.py:37`), not a user-facing key.
- **Provider LLM keys**: read by the LangChain SDK constructors themselves (`ChatOpenAI`
  `model.py:26`, `ChatAnthropic` `:30`, `ChatXAI` `:47`), one per model branch — the key is
  required whenever that model is selected, but none is passed by name to the constructor.
  `OPENAI_API_KEY` is the only one literal in **source**, and only **commented-out** (a
  disabled `OpenAIAdapter` wiring, `route.ts:14`); it and `XAI_API_KEY` also appear in the
  `readme.md` env template (`readme.md` documents OPENAI/TAVILY/XAI/LANGSMITH).
  `ANTHROPIC_API_KEY` has **no occurrence anywhere in the repo** — declared `absent:` in
  frontmatter so gate B3 re-greps it every run and the claim can't silently go stale.

**TS agent** (`agents/typescript/`): same `MODEL`, `GOOGLE_API_KEY` (`model.ts:33`),
`TAVILY_API_KEY` (`search.ts:38`), plus the same implicit provider keys via its `@langchain/*`
constructors.

**External services**: Tavily (search), the four LLM providers (OpenAI/Anthropic/Google/xAI),
optional LangGraph Cloud + LangSmith. The rafinery MCP key (`RAFA_MCP_KEY`) is tooling, not
app — see [the repo toolbox](/brain/rules/repo-toolbox.md).
