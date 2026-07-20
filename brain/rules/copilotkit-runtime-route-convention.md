---
schemaVersion: 1
id: copilotkit-runtime-route-convention
type: convention
domain: agent-bridge
title: The /api/copilotkit route is the runtime bridge between browser and LangGraph
summary: One Next.js route handler proxies the browser to the LangGraph agents; it uses EmptyAdapter (agents own the LLM), gates on a public "api key", and toggles HTTP vs LangGraph-Cloud mode by URL param
links: [agent-name-contract, agent-state-shape-contract, research-chat-flow, env-and-integrations]
cites:
  - src/app/api/copilotkit/route.ts:16 :: EmptyAdapter
  - src/app/api/copilotkit/route.ts:53 :: POST
  - src/app/api/copilotkit/route.ts:69 :: REMOTE_ACTION_URL
  - src/app/api/copilotkit/route.ts:71 :: CopilotRuntime
  - src/app/api/copilotkit/route.ts:99 :: copilotRuntimeNextJSAppRouterEndpoint
  - src/app/page.tsx:28 :: CopilotKit
  - src/app/page.tsx:29 :: runtimeUrl
---
All agent traffic flows through the single App Router POST handler at
`src/app/api/copilotkit/route.ts`. This is where "how does the browser reach the agent" is
answered.

Key conventions to know before touching it:
- **`EmptyAdapter`, not an LLM adapter** (`route.ts:16`). The service adapter is empty
  because the LangGraph agents call their own models (`get_model`), so the runtime is a pure
  proxy. The commented-out `OpenAIAdapter` (`:14-15`) is the alternative if you ever want the
  runtime itself to hold the LLM — don't pattern-match the comment as active.
- **Backend base URL** is `process.env.REMOTE_ACTION_URL` defaulting to
  `http://localhost:8000/copilotkit` (`route.ts:69`) — the local Python/TS agent server. The
  agent-name path segment is appended per [the agent-name contract](/brain/rules/agent-name-contract.md).
- **Two runtime modes.** Default: `LangGraphHttpAgent` pointing at the local/remote agent
  server (`route.ts:71-80`). If `?lgcDeploymentUrl=` is passed, it rebuilds the runtime with
  `LangGraphAgent` against LangGraph Cloud using `LANGSMITH_API_KEY` (`route.ts:82-97`). The
  frontend chooses the mode by appending that param to `runtimeUrl` (`page.tsx:23-25`).
- **Auth is cosmetic.** `isAuthorized` compares an `x-api-key` header to
  `NEXT_PUBLIC_COPILOTKIT_API_KEY` (`route.ts:22-25`) — but that key ships in client JS
  (`page.tsx:33`), so as the code comment states it blocks bots, not real access. Do not
  treat it as a security boundary.
- **SSRF guard** on `lgcDeploymentUrl`: `isSafeDeploymentUrl` (`route.ts:29-51`) rejects
  non-https and private/internal hosts so the proxy (which carries `langsmithApiKey`) can't
  be redirected at internal services.

The provider side: `<CopilotKit runtimeUrl agent headers>` in `page.tsx:28-35` wraps
`<Main/>`. See [the app-shell provider nesting contract](/brain/rules/provider-nesting-contract.md)
for why the wrapping order matters, and [env-and-integrations](/brain/rules/env-and-integrations.md)
for every key this route reads.
