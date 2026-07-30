---
schemaVersion: 1
id: security-posture
type: flow
domain: security
title: "Trust boundaries, auth chokepoints, and secret handling — what is actually exposed"
summary: "Two servers, one thin proxy and one wide-open agent: the Next.js route has a cosmetic x-api-key (the key ships in client JS) and the Python FastAPI agent has NO auth at all, so port 8000 is the real exposure; there is no user identity anywhere, and the only hardened edges are the two SSRF guards on user-supplied URLs"
links: [copilotkit-runtime-route-convention, env-and-integrations, langgraph-agent-convention, build-tooling-convention, research-chat-flow]
absent: add_middleware
absent: use server
absent: getServerSession
absent: next-auth
absent: "@clerk"
absent: useSession
absent: "cookies("
absent: next/headers
absent: Depends
cites:
  - src/app/api/copilotkit/route.ts:86 :: export const POST
  - src/app/api/copilotkit/route.ts:24 :: isAuthorized
  - src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
  - src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
  - src/app/api/copilotkit/route.ts:22 :: visible to anyone via devtools
  - src/app/page.tsx:33 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
  - src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
  - src/app/api/copilotkit/route.ts:19 :: langsmithApiKey
  - agents/python/main.py:17 :: add_langgraph_fastapi_endpoint
  - agents/python/main.py:44 :: 0.0.0.0
  - agents/python/main.py:11 :: load_dotenv
  - agents/python/src/lib/download.py:30 :: _is_safe_url
  - agents/python/src/lib/download.py:41 :: getaddrinfo
  - agents/python/langgraph.json:9 :: .env
  - .gitignore:38 :: .env
description: "Two servers, one thin proxy and one wide-open agent: the Next.js route has a cosmetic x-api-key (the key ships in client JS) and the Python FastAPI agent has NO auth at all, so port 8000 is the real exposure; there is no user identity anywhere, and the only hardened edges are the two SSRF guards on user-supplied URLs"
tags: [security]
timestamp: 2026-07-26T22:44:42.840Z
---
Read this before adding an endpoint, exposing a port, or reasoning about "who can call
this". It is the trust map the code implies — not a findings list.

## The two servers, and which one is the real exposure

**1. The Next.js server — one surface, thin.** The app has exactly ONE server-side entry
point: `POST /api/copilotkit` (`route.ts:86`). There is no `middleware.ts`, no second route
handler, and no Server Action anywhere. **Every absence claimed in this note is
machine-re-checked on every run** — none of them is a claim you have to trust:

| claim | how it stays true |
| --- | --- |
| no Server Action | `absent: use server` in this note's frontmatter — gate B3 re-greps the token in code every run |
| no `middleware.ts` | `coverage.md` inventory row `middleware :: **/middleware.ts :: 0` — `git ls-files` re-counts it every run |
| exactly one route handler | `coverage.md` inventory row `api-routes :: src/app/api/**/route.ts :: 1` — a second handler flips the count and fails the gate |
| no auth middleware on the Python agent | `absent: add_middleware`, `absent: Depends` — the two FastAPI ways to attach a guard; either appearing fails the gate |
| no session/identity anywhere | `absent: getServerSession`, `absent: next-auth`, `absent: "@clerk"`, `absent: useSession` — the provider + hook + server-helper surface |
| nothing reads a cookie or a request header out-of-band | `absent: "cookies("`, `absent: next/headers` — the Next.js server-side request-context import and its cookie accessor |

Those seven `absent:` tokens are the mechanical form of the "no identity" claim below. The
day any of them lands in code, this note fails the gate instead of quietly lying — which is
the point, because a stale *security* absence is the most dangerous kind.

`layout.tsx` is the only Server Component and it
reads no secrets. So the entire server attack surface of the web app is that one handler,
which is a **proxy**: `EmptyAdapter` means it holds no LLM and makes no model calls of its
own ([route convention](/brain/rules/copilotkit-runtime-route-convention.md)).

**2. The Python agent server — wide open.** `main.py` mounts the two agent endpoints via
`add_langgraph_fastapi_endpoint` (`main.py:17,24`) plus a `/health` route, on uvicorn bound
to `0.0.0.0` (`main.py:44`). There is **no auth middleware, no dependency guard, no API key
check** anywhere under `agents/` — `add_middleware` appears nowhere in code (declared
`absent:`, so the checker re-greps it every run). This is the load-bearing fact: anything
that can reach port 8000 can drive the graph directly, bypassing the Next.js route entirely,
and thereby spend your OpenAI/Anthropic/Google/xAI and Tavily quota. It is fine bound to
localhost for `npm run dev`; it is **not** deployable to a public network as-is. If you
expose the agent, the auth has to be added here — there is no other chokepoint behind it.

## Auth chokepoints (all of them)

- `isAuthorized` (`route.ts:24-27`) compares an `x-api-key` header to
  `NEXT_PUBLIC_COPILOTKIT_API_KEY`. The code comment says it outright (`route.ts:21-23`):
  the route is called **from the browser**, so that key ships in client JS
  (`page.tsx:33`) and is visible in devtools. It blocks blind scanners. **It is not access
  control** — never cite it as authorization for anything. Note the flip side, which is an
  availability fact rather than a security one: the check is `Boolean(apiKey) && …`
  (`route.ts:26`), so with `NEXT_PUBLIC_COPILOTKIT_API_KEY` **unset** it denies everyone and
  the route 401s every request silently (`route.ts:87-89`). "Not a security boundary" does
  not make the variable optional — see
  [env-and-integrations](/brain/rules/env-and-integrations.md).
- The Python agent: none (above).
- **There is no user identity in this app at all** — no login, no session, no cookie read,
  no auth provider. This is gated, not asserted: `getServerSession`, `next-auth`, `@clerk`,
  `useSession`, `cookies(` and `next/headers` are each declared `absent:` in this note's
  frontmatter and re-grepped against code every run (`Depends` covers the FastAPI side
  alongside `add_middleware`). Every visitor is the same anonymous actor, and coagent state is
  per-connection, held in LangGraph's in-process `MemorySaver`
  ([build-tooling](/brain/rules/build-tooling-convention.md)) — nothing is persisted or
  partitioned per user. Any feature that needs "this user's data" has to introduce identity
  from scratch; there is nothing to hook into.

## The hardened edges — two SSRF guards on user-supplied URLs

Both places where a *user-controlled URL* becomes a *server-side fetch* are guarded, and
both guards resolve DNS rather than pattern-matching hostnames. Keep it that way:

- `isSafeDeploymentUrl` (`route.ts:59-84`) gates `?lgcDeploymentUrl=`. This matters because
  in that mode the proxy forwards `langsmithApiKey` (`route.ts:19`) to the target — an
  unguarded redirect would leak the LangSmith key to an internal host.
- `_is_safe_url` (`download.py:30`) gates every resource URL the agent downloads, resolving
  via `socket.getaddrinfo` (`download.py:41`) and rejecting if *any* resolved address is
  private/loopback/link-local. Resource URLs come from the model's search results **and**
  from whatever the user types into the Add Resource dialog, so this is genuinely
  attacker-influenced input.

Everything else the servers call is a fixed third-party endpoint (Tavily, the four LLM
providers) — see [env-and-integrations](/brain/rules/env-and-integrations.md).

## Secret handling convention

Names only below — this brain never opens `.env*` and never records a value.

- **Read from the environment, never from a checked-in file.** The web side reads
  `process.env.*` inside `route.ts` only; the Python side reads `os.getenv` in
  `model.py`/`search.py`/`main.py`. `load_dotenv()` (`main.py:11`) hydrates the agent's
  process from a local `.env`, and `langgraph.json` declares `"env": ".env"` (`:9`) for
  `langgraph dev`.
- **`.env` is gitignored** (`.gitignore:38`, plus `.env*.local` at `:29`), as is
  `.claude/settings.local.json` (`:40`) where the `RAFA_MCP_KEY` lives.
- **`NEXT_PUBLIC_` is the browser boundary.** Exactly one env var carries that prefix
  (`NEXT_PUBLIC_COPILOTKIT_API_KEY`) and it is therefore public by construction. Every other
  key — `LANGSMITH_API_KEY`, `TAVILY_API_KEY`, the provider keys — is module-scope on the
  server and must never gain that prefix to "fix" an undefined value in a component.
- Provider keys are mostly **implicit**: the LangChain constructors read them themselves
  ([langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)), so grepping
  for a key name will not find most of them. The authoritative list is
  [env-and-integrations](/brain/rules/env-and-integrations.md), derived per-constructor.

## Prompt/tool surface

The agent binds five no-op tool schemas and acts on the model's tool calls
([research chat flow](/brain/playbooks/research-chat-flow.md)). None of them execute shell,
write files, or hit a database — the only side effects reachable from a model tool call are
an outbound HTTP GET (SSRF-guarded), a Tavily search, and mutations of the in-memory
coagent state. `DeleteResources` is the one destructive action and it is human-gated by a
graph interrupt ([HITL contract](/brain/rules/delete-resources-hitl-contract.md)). That
narrowness is why the missing agent-server auth is a quota/abuse problem rather than an RCE
one — but it is the property to re-check whenever a tool is added.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/api/copilotkit/route.ts:86](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L86) — `export const POST`
[2] [src/app/api/copilotkit/route.ts:24](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L24) — `isAuthorized`
[3] [src/app/api/copilotkit/route.ts:26](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L26) — `Boolean(apiKey)`
[4] [src/app/api/copilotkit/route.ts:87-89](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L87-L89) — `Unauthorized`
[5] [src/app/api/copilotkit/route.ts:22](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L22) — `visible to anyone via devtools`
[6] [src/app/page.tsx:33](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/page.tsx#L33) — `NEXT_PUBLIC_COPILOTKIT_API_KEY`
[7] [src/app/api/copilotkit/route.ts:59](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L59) — `isSafeDeploymentUrl`
[8] [src/app/api/copilotkit/route.ts:19](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L19) — `langsmithApiKey`
[9] [agents/python/main.py:17](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L17) — `add_langgraph_fastapi_endpoint`
[10] [agents/python/main.py:44](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L44) — `0.0.0.0`
[11] [agents/python/main.py:11](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/main.py#L11) — `load_dotenv`
[12] [agents/python/src/lib/download.py:30](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L30) — `_is_safe_url`
[13] [agents/python/src/lib/download.py:41](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/download.py#L41) — `getaddrinfo`
[14] [agents/python/langgraph.json:9](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/langgraph.json#L9) — `.env`
[15] [.gitignore:38](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/.gitignore#L38) — `.env`

<!-- okf:citations:end -->
