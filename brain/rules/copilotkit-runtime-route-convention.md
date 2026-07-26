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
  - src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
  - src/app/api/copilotkit/route.ts:74 :: dns.lookup
  - src/app/page.tsx:28 :: CopilotKit
  - src/app/page.tsx:29 :: runtimeUrl
  - src/app/api/copilotkit/route.ts:26 :: Boolean(apiKey)
  - src/app/api/copilotkit/route.ts:87-89 :: Unauthorized
---
All agent traffic flows through the single App Router POST handler at
`src/app/api/copilotkit/route.ts`. This is where "how does the browser reach the agent" is
answered.

Key conventions to know before touching it:
- **`EmptyAdapter`, not an LLM adapter** (`route.ts:18`). The service adapter is empty
  because the LangGraph agents call their own models (`get_model`), so the runtime is a pure
  proxy. The commented-out `OpenAIAdapter` (`:16-17`) is the alternative if you ever want the
  runtime itself to hold the LLM — don't pattern-match the comment as active.
- **Backend base URL** is `process.env.REMOTE_ACTION_URL` defaulting to
  `http://localhost:8000/copilotkit` (`route.ts:104-105`) — the local Python/TS agent server.
  The agent-name path segment is appended per
  [the agent-name contract](/brain/rules/agent-name-contract.md).
- **Two runtime modes.** Default: `LangGraphHttpAgent` pointing at the local/remote agent
  server (`route.ts:107-116`). If `?lgcDeploymentUrl=` is passed, it rebuilds the runtime with
  `LangGraphAgent` against LangGraph Cloud using `LANGSMITH_API_KEY` (`route.ts:118-133`). The
  frontend chooses the mode by appending that param to `runtimeUrl` (`page.tsx:23-25`).
- **Auth is cosmetic as a boundary, but the key is MANDATORY to run.** Two separate facts,
  and missing the second is the most common way this app appears "broken":
  - *Boundary:* `isAuthorized` compares an `x-api-key` header to
    `NEXT_PUBLIC_COPILOTKIT_API_KEY` (`route.ts:24-27`) — but that key ships in client JS
    (`page.tsx:33`), so as the code comment states it blocks bots, not real access. Never
    treat it as a security boundary.
  - *Runtime requirement:* the check is `Boolean(apiKey) && …` (`route.ts:26`), so with the
    var **unset** it is false for every caller and this handler 401s every request
    (`route.ts:87-89`). Nothing surfaces the reason — the chat just never responds. "Weak
    gate" does **not** mean "optional to configure": see
    [env-and-integrations](/brain/rules/env-and-integrations.md) for the full consequence.

  This route is the app's ONLY server-side entry point —
  see [the security posture](/brain/playbooks/security-posture.md).
- **SSRF guard** on `lgcDeploymentUrl`: `isSafeDeploymentUrl` (`route.ts:59-84`) is `async`
  and **resolves DNS** (`dns.lookup(..., { all: true })`, `route.ts:74`) before trusting the
  URL — it rejects non-https, `localhost`/`.local`, and any host whose *resolved* addresses
  include a private/loopback/link-local/multicast IPv4 or IPv6 (`isPrivateIPv4` `:29`,
  `isPrivateIPv6` `:43`), so the proxy (which carries `langsmithApiKey`) can't be redirected
  at internal services. Because it awaits DNS, `POST` must `await` it (`route.ts:93-101`).
  *(Historical note: this guard used to be a hostname-string check only; it was hardened to
  match `download.py`'s `_is_safe_url` — don't "simplify" it back to string matching.)*

The provider side: `<CopilotKit runtimeUrl agent headers>` in `page.tsx:28-35` wraps
`<Main/>`. See [the app-shell provider nesting contract](/brain/rules/provider-nesting-contract.md)
for why the wrapping order matters, and [env-and-integrations](/brain/rules/env-and-integrations.md)
for every key this route reads.
