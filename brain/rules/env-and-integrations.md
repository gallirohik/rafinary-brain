---
schemaVersion: 1
id: env-and-integrations
type: convention
domain: external-integrations
title: Environment variables and external services this app reads
summary: NEXT_PUBLIC_COPILOTKIT_API_KEY is REQUIRED for the app to function — unset means every POST /api/copilotkit 401s silently; beyond that the runtime route reads LangSmith/deployment keys, each agent reads MODEL + provider keys + TAVILY_API_KEY; OPENAI_API_KEY appears only commented-out in the runtime route while provider-specific keys (Anthropic/xAI/etc.) are read implicitly by the LangChain constructors in this repo's agent code. ANTHROPIC_API_KEY is not present in the cited files; a repo-wide grep is required to prove global absence.
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
timestamp: 2026-08-01T10:25:28.000Z
---

Env var names and where source reads them (values never inspected; `.env*` never opened).

Summary (durable claims, with cites):
- NEXT_PUBLIC_COPILOTKIT_API_KEY — runtime route uses process.env.NEXT_PUBLIC_COPILOTKIT_API_KEY and the isAuthorized guard returns Boolean(apiKey) && req.headers.get("x-api-key") === apiKey (src/app/api/copilotkit/route.ts:16-26). When that env var is unset the left conjunct is false and POST /api/copilotkit returns 401 (src/app/api/copilotkit/route.ts:87-89). The frontend sends the same header from process.env.NEXT_PUBLIC_COPILOTKIT_API_KEY (src/app/page.tsx:30-36), so a fresh clone without that env var makes the chat effectively non-functional (silent failure mode).

- LANGSMITH_API_KEY / LGC_DEPLOYMENT_URL / REMOTE_ACTION_URL — the runtime route reads process.env.LANGSMITH_API_KEY and accepts a ?lgcDeploymentUrl= param with a fallback to process.env.LGC_DEPLOYMENT_URL; the backend base URL falls back to process.env.REMOTE_ACTION_URL || "http://localhost:8000/copilotkit" (src/app/api/copilotkit/route.ts:100-120).

- OPENAI_API_KEY — only present as a commented-out line in the runtime route ("// const openai = new OpenAI({ apiKey: process.env.OPENAI_API_KEY });") so no active wiring exists in the runtime route to an OPENAI_API_KEY (src/app/api/copilotkit/route.ts:16).

Agent / provider keys and behavior (durable patterns):
- Python agent (agents/python/src/lib/model.py): MODEL is read via os.getenv("MODEL", state_model) and used to select the provider. For the google_genai branch the code explicitly calls os.getenv("GOOGLE_API_KEY") and passes it to ChatGoogleGenerativeAI(api_key=... ) (agents/python/src/lib/model.py:24-36). For the anthropic branch the code instantiates ChatAnthropic(...) without any explicit os.getenv("ANTHROPIC_API_KEY") in this file — the LangChain provider's constructor is relied upon to obtain credentials implicitly (agents/python/src/lib/model.py:24-36).

- TypeScript agent (agents/typescript/src/model.ts): same pattern — process.env.MODEL drives selection, ChatGoogleGenerativeAI is constructed with apiKey: process.env.GOOGLE_API_KEY || undefined, while ChatAnthropic is constructed without an explicit process.env token in this file (agents/typescript/src/model.ts:1-44).

- Tavily (search) keys: agents/python/src/lib/search.py reads tavily_api_key = os.getenv("TAVILY_API_KEY") and builds TavilyClient(api_key=tavily_api_key) (agents/python/src/lib/search.py:1-80). The TypeScript search node builds tavily({ apiKey: process.env.TAVILY_API_KEY }) (agents/typescript/src/search.ts:1-80).

Durable conclusion and verification note about ANTHROPIC_API_KEY:
- In the files cited above, ChatAnthropic is instantiated without an explicit os.getenv/process.env call for ANTHROPIC_API_KEY (agents/python/src/lib/model.py:24-36; agents/typescript/src/model.ts:1-44). This establishes the important operational point: provider credentials for Anthropic (and other providers like xAI/grok) are obtained implicitly via the provider SDK constructors, not by a named env var referenced in the agent files.

- However, the original note's frontmatter claim that ANTHROPIC_API_KEY has "no occurrence anywhere in this repository" is a repository-wide absence claim that cannot be proven by inspecting the cited files alone. I did NOT run a repository-wide textual search (grep) in this adjudication step. To make the absence claim durable and mechanically verifiable, perform one of the following and update this note accordingly:
  - Run a repo-wide grep or platform verify-citations re-grep for the literal token ANTHROPIC_API_KEY. If the token is not found anywhere in the repository at the merge sha, keep absent: ANTHROPIC_API_KEY and add a verification permalink (file:line not found).
  - If the token is found, update this note's cites to include the file:line where it appears and record whether the code actively reads it or only documents it.

Why this rewrite: the brain should preserve the durable, non-volatile guidance (what to set to get a working runtime, which files read which named keys, the implicit-constructor trap). It should not assert an absolute repository-wide absence without evidence of a full-text search. The rewritten note above keeps the durable claims and the absent: field as a reminder to re-verify; it documents the exact lines that prove the key behaviors and instructs the next operator to run the grep that closes the remaining uncertainty.

Action items (to close the verification gap):
- Run: git grep -n "ANTHROPIC_API_KEY" -- || true (or platform verify-citations re-grep) at sha 1ebb712b4a31969dd2ade1b59f92b18d1f456033. If not found, mark the absence verified and leave a short verification line with date and command result. If found, update the note with the new cite(s).

Cited evidence (merge sha 1ebb712b4a31969dd2ade1b59f92b18d1f456033):
- src/app/api/copilotkit/route.ts:16-26 (isAuthorized, NEXT_PUBLIC_COPILOTKIT_API_KEY)
- src/app/api/copilotkit/route.ts:87-89 (Unauthorized response when not authorized)
- src/app/page.tsx:30-36 (frontend sends x-api-key using NEXT_PUBLIC_COPILOTKIT_API_KEY)
- src/app/api/copilotkit/route.ts:100-120 (LGC_DEPLOYMENT_URL and REMOTE_ACTION_URL usage)
- agents/python/src/lib/model.py:24-36 (MODEL selection; ChatAnthropic constructed with no explicit ANTHROPIC_API_KEY read; ChatGoogleGenerativeAI uses os.getenv("GOOGLE_API_KEY"))
- agents/python/src/lib/search.py:1-80 (tavily_api_key = os.getenv("TAVILY_API_KEY"))
- agents/typescript/src/model.ts:1-44 (MODEL selection; ChatAnthropic constructed without explicit ANTHROPIC env var; ChatGoogleGenerativeAI uses process.env.GOOGLE_API_KEY)
- agents/typescript/src/search.ts:1-80 (tavily client uses process.env.TAVILY_API_KEY)

Links: keep existing brain links (copilotkit-runtime-route-convention, langgraph-agent-convention, repo-toolbox, security-posture).

If you want, I will run a repo-wide grep for the token ANTHROPIC_API_KEY now and update this note to mark the absence verified (or include the file:line where it appears).