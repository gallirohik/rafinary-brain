---
schemaVersion: 1
id: security-posture
type: flow
domain: security
title: "Trust boundaries, auth chokepoints, and secret handling — what is actually exposed"
summary: "Two servers, one thin proxy and one wide-open agent: the Next.js route uses a cosmetic x-api-key (the key ships in client JS) and the Python FastAPI agent has NO auth in main.py, so port 8000 is the real exposure; there is no user identity anywhere. Both SSRF guards resolve DNS (deny-private), and the Next.js guard additionally enforces a configured allowed host when LGC_DEPLOYMENT_URL is set — update: this prevents arbitrary public-host exfiltration unless the deployment is misconfigured."
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
  - src/app/api/copilotkit/route.ts:19 :: langsmithApiKey
  - src/app/api/copilotkit/route.ts:21-27 :: isAuthorized
  - src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
  - src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
  - src/app/api/copilotkit/route.ts:81 :: addresses.every
  - src/app/api/copilotkit/route.ts:86 :: export const POST
  - src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
  - src/app/api/copilotkit/route.ts:122-123 :: LangGraphAgent({ deploymentUrl, langsmithApiKey })
  - src/app/page.tsx:24 :: lgcDeploymentUrl
  - src/app/page.tsx:33 :: NEXT_PUBLIC_COPILOTKIT_API_KEY
  - src/lib/model-selector-provider.tsx:40 :: lgcDeploymentUrl
  - agents/python/main.py:11 :: load_dotenv
  - agents/python/main.py:17 :: add_langgraph_fastapi_endpoint
  - agents/python/main.py:44 :: 0.0.0.0
  - agents/python/src/lib/download.py:30 :: _is_safe_url
  - agents/python/src/lib/download.py:41 :: getaddrinfo
  - agents/python/langgraph.json:9 :: .env
  - .gitignore:38 :: .env
description: "Read this before adding an endpoint, exposing a port, or reasoning about 'who can call this'. It is the trust map the code implies — not a findings list. This revision updates the original residual finding about ?lgcDeploymentUrl exfiltration to match the merged code's allow-known host check.

## The two servers, and which one is the real exposure

1) The Next.js server — one surface, thin.

- The app's single server-side entry used by the UI is POST /api/copilotkit (export const POST) — see src/app/api/copilotkit/route.ts:86.
- The route implements a client-visible x-api-key check; the check reads NEXT_PUBLIC_COPILOTKIT_API_KEY and compares the request header (isAuthorized) — see src/app/api/copilotkit/route.ts:21-27 and the 401 return at src/app/api/copilotkit/route.ts:87-89. The page populates the header from process.env.NEXT_PUBLIC_COPILOTKIT_API_KEY (src/app/page.tsx:33), so that value is intentionally public and should not be treated as an access-control secret.
- The client can supply a ?lgcDeploymentUrl via the UI (model-selector-provider → page.tsx:24 and src/lib/model-selector-provider.tsx:40), which the route may accept after the server-side check below.
- The route constructs runtime agents and — if a deploymentUrl is set — passes deploymentUrl and langsmithApiKey into the LangGraphAgent constructors (src/app/api/copilotkit/route.ts:122-123). That means any host the server connects to on behalf of a request may receive LANGSMITH_API_KEY if the route is configured to use a caller-provided deploymentUrl.

2) The Python agent server — wide open (unless you add auth there).

- main.py mounts langgraph FastAPI endpoints via add_langgraph_fastapi_endpoint and starts uvicorn (agents/python/main.py:17 and agents/python/main.py:44). load_dotenv is used to hydrate local env (agents/python/main.py:11) and the langgraph dev tooling declares an env file (agents/python/langgraph.json:9). There is no auth middleware configured in main.py; any client that can reach the bound host (0.0.0.0 if deployed as-is) can call the agent endpoints directly. This is the primary deployment chokepoint.

## Auth chokepoints (all of them)

- Next.js route's isAuthorized (src/app/api/copilotkit/route.ts:21-27) compares the request header to NEXT_PUBLIC_COPILOTKIT_API_KEY and returns 401 otherwise (src/app/api/copilotkit/route.ts:87-89). That header value is shipped to the browser (src/app/page.tsx:33) and therefore is not an authorization boundary — it only blocks basic scanners.
- The Python agent exposes endpoints directly in main.py (agents/python/main.py:17, agents/python/main.py:44) and has no auth guard in the merged code.
- There is no user identity (sessions, providers, server session helpers) in the codebase as relied-on absence tokens indicate; any feature needing per-user data must add identity.

## The guarded edges — DNS-resolution deny-private checks (and an allow-known host for the proxy)

- Next.js: isSafeDeploymentUrl resolves DNS and rejects private/loopback/link-local addresses; it also requires the parsed hostname to equal the configured LGC_DEPLOYMENT_URL host when that env var is present (see src/app/api/copilotkit/route.ts:59 and the address checks at src/app/api/copilotkit/route.ts:81). The code computes allowedDeploymentHost from process.env.LGC_DEPLOYMENT_URL and explicitly compares hostnames before proceeding; this is an allow-known host check when LGC_DEPLOYMENT_URL is configured.
- Python agent: _is_safe_url does an equivalent DNS-resolution-based check using socket.getaddrinfo and ipaddress checks (agents/python/src/lib/download.py:30 and agents/python/src/lib/download.py:41), rejecting names that resolve to private/loopback/link-local/multicast/reserved addresses.

### Residual — updated: ?lgcDeploymentUrl exfiltration is mitigated by an allow-known host check

- The route forwards LANGSMITH_API_KEY into LangGraphAgent when a deploymentUrl is used (src/app/api/copilotkit/route.ts:122-123). That remains true: any deploymentUrl the server accepts becomes the target that can receive the key.
- However, the merged Next.js guard now requires the requested hostname to match the configured LGC_DEPLOYMENT_URL host (the code computes allowedDeploymentHost from process.env.LGC_DEPLOYMENT_URL and enforces hostname === allowedDeploymentHost inside isSafeDeploymentUrl — see src/app/api/copilotkit/route.ts:59). Therefore arbitrary public hosts named by callers do not pass validation unless the server is misconfigured.
- The remaining ways this could still leak are configuration faults or operational missteps:
  - If LGC_DEPLOYMENT_URL is set to an attacker-controllable host (i.e., the deployment environment is compromised), the server would accept that host and forward LANGSMITH_API_KEY. (The host equality check uses server env and is not influenced by request params.)
  - If LANGSMITH_API_KEY is set in a network-reachable environment without locking down access to the agent endpoints (agents/python/main.py binding to 0.0.0.0), an attacker reaching port 8000 can use the agent directly and consume provider quota.

Treat the risk as P0 if you ever set LANGSMITH_API_KEY in a network-exposed deployment; otherwise it is latent.

## Secret handling convention (durable points)

- Server-only secrets are read from process.env or os.getenv; the Next.js route reads process.env.* in src/app/api/copilotkit/route.ts (langsmithApiKey at src/app/api/copilotkit/route.ts:19), and the Python side uses load_dotenv plus os.getenv (agents/python/main.py:11). The brain never records secret values.
- .env is ignored by git (.gitignore:38) and langgraph dev tooling declares env: ".env" (agents/python/langgraph.json:9).
- NEXT_PUBLIC_ prefix is the browser boundary; NEXT_PUBLIC_COPILOTKIT_API_KEY is intentionally public (src/app/page.tsx:33).

## Operational guidance (actionable)

- Do not set LANGSMITH_API_KEY in any environment where the Python agent is reachable by untrusted networks unless you add auth middleware to the Python server or otherwise restrict access (agents/python/main.py:17 and agents/python/main.py:44).
- Set LGC_DEPLOYMENT_URL to the single legitimate host for LangGraph deployments and keep that env var out of attacker control — the Next.js route enforces hostname equality to that value before using a caller-provided lgcDeploymentUrl (src/app/api/copilotkit/route.ts:59 and src/app/api/copilotkit/route.ts:122-123).
- Treat NEXT_PUBLIC_COPILOTKIT_API_KEY as a nuisance anti-scan token (client-visible); do not rely on it for authentication (src/app/api/copilotkit/route.ts:21-27 and src/app/page.tsx:33).

<!-- okf:citations:start -->

# Citations

[1] src/app/api/copilotkit/route.ts:19 — `langsmithApiKey`
[2] src/app/api/copilotkit/route.ts:21-27 — `isAuthorized` / `Boolean(apiKey)`
[3] src/app/api/copilotkit/route.ts:59 — `isSafeDeploymentUrl` (DNS-resolution + host checks)
[4] src/app/api/copilotkit/route.ts:81 — `addresses.every` (deny-private resolution check)
[5] src/app/api/copilotkit/route.ts:86 — `export const POST`
[6] src/app/api/copilotkit/route.ts:87-89 — `Unauthorized` (401)
[7] src/app/api/copilotkit/route.ts:122-123 — `LangGraphAgent({ deploymentUrl, langsmithApiKey })`
[8] src/app/page.tsx:24 — `lgcDeploymentUrl` read from client URL
[9] src/app/page.tsx:33 — `NEXT_PUBLIC_COPILOTKIT_API_KEY` used in client header
[10] src/lib/model-selector-provider.tsx:40 — `lgcDeploymentUrl` read from window.location
[11] agents/python/main.py:11 — `load_dotenv()`
[12] agents/python/main.py:17 — `add_langgraph_fastapi_endpoint`
[13] agents/python/main.py:44 — `0.0.0.0` (uvicorn host)
[14] agents/python/src/lib/download.py:30 — `_is_safe_url`
[15] agents/python/src/lib/download.py:41 — `socket.getaddrinfo` used via asyncio executor
[16] agents/python/langgraph.json:9 — `"env": ".env"`
[17] .gitignore:38 — `.env` is ignored

<!-- okf:citations:end -->