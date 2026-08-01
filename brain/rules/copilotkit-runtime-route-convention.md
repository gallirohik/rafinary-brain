---
schemaVersion: 1
id: copilotkit-runtime-route-convention
type: convention
domain: agent-bridge
title: The /api/copilotkit route is the runtime bridge between browser and LangGraph
summary: One Next.js route handler proxies the browser to the LangGraph agents; it uses EmptyAdapter (agents own the LLM), gates on a public "api key", toggles HTTP vs LangGraph-Cloud mode by URL param, and DNS-resolves the lgcDeploymentUrl before trusting it
links: [agent-name-contract, agent-state-shape-contract, research-chat-flow, env-and-integrations, security-posture]
cites:
  - src/app/api/copilotkit/route.ts:18 :: EmptyAdapter
  - src/app/api/copilotkit/route.ts:86 :: POST
  - src/app/api/copilotkit/route.ts:105 :: REMOTE_ACTION_URL
  - src/app/api/copilotkit/route.ts:107 :: CopilotRuntime
  - src/app/api/copilotkit/route.ts:135 :: copilotRuntimeNextJSAppRouterEndpoint
  - src/app/api/copilotkit/route.ts:59 :: allowedDeploymentHost (LGC_DEPLOYMENT_URL)
  - src/app/api/copilotkit/route.ts:74 :: dns.lookup
  - src/app/page.tsx:28 :: CopilotKit
  - src/app/page.tsx:29 :: runtimeUrl
  - src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
  - src/app/api/copilotkit/route.ts:87-89 :: Unauthorized / isSafeDeploymentUrl host-check
description: "One Next.js route handler proxies the browser to the LangGraph agents; it uses EmptyAdapter (agents own the LLM), gates on a public 'api key', toggles HTTP vs LangGraph-Cloud mode by URL param, and DNS-resolves the lgcDeploymentUrl before trusting it"
tags: [agent-bridge]
timestamp: 2026-08-01T05:37:23.000Z
---

What this file answers: how the browser reaches agents and what the runtime will proxy.

Overview and durable contracts
- Single entry point: the browser talks to a single App Router POST handler in src/app/api/copilotkit/route.ts (export const POST = async (req: NextRequest) => { ... }) — all agent traffic from the client-side runtimeUrl flows through this handler (src/app/api/copilotkit/route.ts:100-112).

- EmptyAdapter: the runtime does not embed a model adapter — it uses EmptyAdapter so agents call their own models (llmAdapter construction): "const llmAdapter = new EmptyAdapter();" (src/app/api/copilotkit/route.ts:18). Do not assume the runtime performs LLM inference unless you intentionally swap adapters.

- Backend base URL: the local/remote HTTP agent server base is REMOTE_ACTION_URL and falls back to http://localhost:8000/copilotkit when not set: "const baseUrl = process.env.REMOTE_ACTION_URL || \"http://localhost:8000/copilotkit\";" (src/app/api/copilotkit/route.ts:104-105). The agent-name segment is appended when the runtime constructs LangGraphHttpAgent URLs.

- Two runtime modes (how the param selects them):
  - Default / HTTP (local or remote agent server): the runtime is constructed with LangGraphHttpAgent instances that proxy to the REMOTE_ACTION_URL base (see the initial CopilotRuntime creation) (src/app/api/copilotkit/route.ts:100-120).
  - LangGraph Cloud mode: when a deploymentUrl is present (either passed as ?lgcDeploymentUrl= in the client URL or via the LGC_DEPLOYMENT_URL env fallback) the runtime is rebuilt to use LangGraphAgent and the runtime receives the LANGSMITH_API_KEY: "research_agent: new LangGraphAgent({ deploymentUrl, langsmithApiKey, graphId: \"research_agent\" })" (src/app/api/copilotkit/route.ts:129-136).
  The client constructs runtimeUrl as either `/api/copilotkit` or `/api/copilotkit?lgcDeploymentUrl=...` and sends the x-api-key header (page.tsx:22-34).

- Authorization gate (cosmetic boundary + runtime requirement):
  - Cosmetic boundary: the handler checks an x-api-key header against NEXT_PUBLIC_COPILOTKIT_API_KEY and returns 401 if missing/incorrect: "return Boolean(apiKey) && req.headers.get(\"x-api-key\") === apiKey;" and the POST handler returns Unauthorized when isAuthorized fails (src/app/api/copilotkit/route.ts:24-27 and src/app/api/copilotkit/route.ts:100-112). Because this is the NEXT_PUBLIC_ prefixed env var it is exposed to client JS (page.tsx), so treat this only as a weak throttle against blind bots, not as a secret.
  - Runtime requirement: the code explicitly requires the presence of the API key check (Boolean(apiKey) is part of the function), so if the NEXT_PUBLIC_COPILOTKIT_API_KEY env var is unset the handler will 401 every request — the app will appear to ‘not respond’ (src/app/api/copilotkit/route.ts:26 and src/app/api/copilotkit/route.ts:100-112).

- SSRF / deployment-url guard (accurate, updated view):
  - The handler performs a multi-step validation of any caller-supplied lgcDeploymentUrl. First it parses the URL and rejects non-https and explicit local hostnames: "if (parsed.protocol !== \"https:\") return false;" and "if (hostname === \"localhost\" || hostname.endsWith(\".local\")) return false;" (src/app/api/copilotkit/route.ts:81-88).
  - It DNS-resolves the hostname and checks every resolved address against private/loopback/link-local rules via isPrivateIPv4/isPrivateIPv6 helpers — the code calls dns.lookup(..., { all: true, verbatim: true }) and then ensures addresses.every(...) are not private (src/app/api/copilotkit/route.ts:74 and src/app/api/copilotkit/route.ts:100-112).
  - Important correction vs older wording: the function also enforces that, when the server has LGC_DEPLOYMENT_URL configured, the requested hostname must exactly match that configured host. The code computes an allowedDeploymentHost from process.env.LGC_DEPLOYMENT_URL and returns false if it is unset or the hostname differs: "const configured = process.env.LGC_DEPLOYMENT_URL; ... return new URL(configured).hostname.toLowerCase();" and "if (!allowedDeploymentHost || hostname !== allowedDeploymentHost) return false;" (src/app/api/copilotkit/route.ts:59 and src/app/api/copilotkit/route.ts:87-89). In short: when LGC_DEPLOYMENT_URL is set, the handler performs an exact-host allowlist in addition to the DNS-based deny-private checks. If LGC_DEPLOYMENT_URL is unset, caller-supplied lgcDeploymentUrl is rejected by this code path (i.e. the request param won't be allowed), and the code will fall back to using process.env.LGC_DEPLOYMENT_URL when building the runtime.

  This means the earlier claim that the guard is only a "deny-private" check that leaves an open path for arbitrary public hosts to receive langsmithApiKey is incorrect. The runtime will only hand langsmithApiKey to a deploymentUrl that passed isSafeDeploymentUrl; that routine requires an exact hostname match when the env is configured (src/app/api/copilotkit/route.ts:59 and src/app/api/copilotkit/route.ts:87-89). If you want to tighten further, maintainers should keep the exact-host check (already present) and consider an explicit allowlist or removal of passing the langsmithApiKey at runtime.

Provider-side/Client contract
- The client wraps the UI with <CopilotKit runtimeUrl={runtimeUrl} headers={{ "x-api-key": process.env.NEXT_PUBLIC_COPILOTKIT_API_KEY || "" }} />; the public api key is intentionally exposed to client-side JS (src/app/page.tsx:22-34). That is why the server-side check is a weak gate, and why production deployments must ensure NEXT_PUBLIC_COPILOTKIT_API_KEY is set if you want the gate in place.

Operational notes (durable warnings)
- Do not treat the x-api-key check as a security boundary. It is visible in client bundles and only blocks unsophisticated scanners (src/app/api/copilotkit/route.ts:24-27 and src/app/page.tsx:28-34).
- Do not remove the dns.resolve + private-IP checks (dns.lookup + isPrivateIPv4/isPrivateIPv6) — they prevent redirection to internal services (src/app/api/copilotkit/route.ts:74 and src/app/api/copilotkit/route.ts:100-112).
- The code already requires exact-host matching against LGC_DEPLOYMENT_URL when configured. If you need stricter controls for langsmithApiKey exposure, add an explicit allowlist, remove runtime passing of langsmithApiKey, or broker LangGraph Cloud access from a separate backend service (see src/app/api/copilotkit/route.ts:59 and src/app/api/copilotkit/route.ts:129-136).

Minimal code anchors (evidence pointers)
- EmptyAdapter used: src/app/api/copilotkit/route.ts:18 :: "const llmAdapter = new EmptyAdapter();"
- Public key check: src/app/api/copilotkit/route.ts:24-27 :: "const apiKey = process.env.NEXT_PUBLIC_COPILOTKIT_API_KEY; return Boolean(apiKey) && req.headers.get(\"x-api-key\") === apiKey;"
- POST handler auth and lgcDeploymentUrl check: src/app/api/copilotkit/route.ts:100-112 :: "if (!isAuthorized(req)) { return NextResponse.json({ error: \"Unauthorized\" }, { status: 401 }); } ... if (requestedDeploymentUrl && !(await isSafeDeploymentUrl(requestedDeploymentUrl))) { return NextResponse.json({ error: \"Invalid lgcDeploymentUrl\" }, { status: 400 }); }"
- allowedDeploymentHost derivation (LGC_DEPLOYMENT_URL): src/app/api/copilotkit/route.ts:59 :: "const configured = process.env.LGC_DEPLOYMENT_URL; ... return new URL(configured).hostname.toLowerCase();"
- exact-host check in isSafeDeploymentUrl: src/app/api/copilotkit/route.ts:87-89 :: "if (!allowedDeploymentHost || hostname !== allowedDeploymentHost) return false;"
- DNS lookup and private-IP checks: src/app/api/copilotkit/route.ts:74 and src/app/api/copilotkit/route.ts:100-112 :: "await dns.lookup(hostname, { all: true, verbatim: true })" and address filtering using isPrivateIPv4/isPrivateIPv6
- LangGraph Cloud runtime construction with LANGSMITH_API_KEY: src/app/api/copilotkit/route.ts:129-136 :: "research_agent: new LangGraphAgent({ deploymentUrl, langsmithApiKey, graphId: \"research_agent\" })"

See the cited lines for the exact code; this rewrite preserves the durable guidance and corrects the previous note's overstatement about the guard being purely deny-private.

<!-- okf:citations:start -->

# Citations

[1] src/app/api/copilotkit/route.ts:18 — const llmAdapter = new EmptyAdapter();
[2] src/app/api/copilotkit/route.ts:24-27 — NEXT_PUBLIC_COPILOTKIT_API_KEY check / Boolean(apiKey)
[3] src/app/api/copilotkit/route.ts:59 — allowedDeploymentHost derived from process.env.LGC_DEPLOYMENT_URL
[4] src/app/api/copilotkit/route.ts:74 — dns.lookup(hostname, { all: true, verbatim: true })
[5] src/app/api/copilotkit/route.ts:87-89 — exact-host check: if (!allowedDeploymentHost || hostname !== allowedDeploymentHost) return false;
[6] src/app/api/copilotkit/route.ts:100-112 — POST handler auth, await isSafeDeploymentUrl, and baseUrl (REMOTE_ACTION_URL fallback)
[7] src/app/api/copilotkit/route.ts:129-136 — CopilotRuntime rebuilt with LangGraphAgent({ deploymentUrl, langsmithApiKey })
[8] src/app/page.tsx:22-34 — client constructs runtimeUrl and exposes NEXT_PUBLIC_COPILOTKIT_API_KEY in headers

<!-- okf:citations:end -->