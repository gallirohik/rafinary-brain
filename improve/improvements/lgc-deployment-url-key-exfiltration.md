---
id: lgc-deployment-url-key-exfiltration
type: Improvement
schemaVersion: 1
priority: P1
category: security
status: open
title: The SSRF guard blocks private targets but not public ones — any caller can redirect the proxy at their own host and receive LANGSMITH_API_KEY
summary: "?lgcDeploymentUrl= is read straight from the request URL and, when set, becomes the LangGraph SDK client's apiUrl with langsmithApiKey as its apiKey; isSafeDeploymentUrl only rejects hosts resolving to private addresses, so https://attacker.example passes and the server sends the LangSmith key to it — and the x-api-key gate in front of it is documented as not being access control"
fix: "Allowlist the deployment host instead of only denying private ones — compare the parsed hostname against LGC_DEPLOYMENT_URL's host (or an explicit env allowlist) and 400 on anything else; while in page.tsx, wrap the value in encodeURIComponent (~15 min)"
leverage: { impact: high, effort: low }
blast_radius: [agent-bridge, external-integrations, security]
cites:
  - src/app/api/copilotkit/route.ts:92 :: lgcDeploymentUrl
  - src/app/api/copilotkit/route.ts:19 :: LANGSMITH_API_KEY
  - src/app/api/copilotkit/route.ts:122 :: deploymentUrl
  - src/app/api/copilotkit/route.ts:123 :: langsmithApiKey
  - src/app/api/copilotkit/route.ts:81 :: addresses.every
  - src/app/api/copilotkit/route.ts:22 :: visible to anyone via devtools
  - src/lib/model-selector-provider.tsx:40 :: lgcDeploymentUrl
  - src/app/page.tsx:24 :: lgcDeploymentUrl
found: 2026-07-30
description: "?lgcDeploymentUrl= is read straight from the request URL and, when set, becomes the LangGraph SDK client's apiUrl with langsmithApiKey as its apiKey; isSafeDeploymentUrl only rejects hosts resolving to private addresses, so https://attacker.example passes and the server sends the LangSmith key to it — and the x-api-key gate in front of it is documented as not being access control"
tags: [security, P1]
timestamp: 2026-07-30
---
This is the residual the SSRF fix
([lgc-deployment-url-ssrf-asymmetry](/improve/improvements/lgc-deployment-url-ssrf-asymmetry.md))
did not close: the guard is a *deny-private* check, not an *allow-known* one, so every public
host on the internet still passes.

**Brain coverage — corrected 2026-08-01.** This row originally recorded that the brain did
*not* name the residual: `security-posture` called `isSafeDeploymentUrl` a "hardened edge" and
framed the risk as leaking "the LangSmith key to an **internal** host", which is only half the
threat model. That gap was raised as a scan major and closed —
[security-posture](/brain/playbooks/security-posture.md) now heads the section
"deny-private, not allow-known", states per guard what it covers and does not, and carries a
"Residual — not closed" subsection naming this row with the full path. **The code is
unchanged**, so this row stays `open` at P1; only the brain's description of it moved.

## The path, end to end

1. `lgcDeploymentUrl` originates in the **browser's own query string**
   (`model-selector-provider.tsx:40`) and is appended to the runtime URL by `page.tsx:24`, so
   it is fully caller-controlled — either by sending someone a crafted link, or by calling
   `POST /api/copilotkit?lgcDeploymentUrl=…` directly.
2. The route reads it from `searchParams` (`route.ts:92`) and runs `isSafeDeploymentUrl`.
   That guard's terminal test is `addresses.every(addr => !isPrivate…)` (`route.ts:81`) —
   it returns **true** for any hostname resolving to a normal public IP.
3. It then becomes `deploymentUrl` on `LangGraphAgent` alongside `langsmithApiKey`
   (`route.ts:122-123`). The vendored SDK (`@ag-ui/langgraph@0.0.42`, `dist/index.mjs`)
   constructs its `Client` from those two config fields — the caller-supplied
   `deploymentUrl` becomes the client's base URL, and `langsmithApiKey` becomes its
   credential — so the key rides the outbound request to whatever host was supplied.

The gate in front of this is not a gate: `isAuthorized` compares `x-api-key` to
`NEXT_PUBLIC_COPILOTKIT_API_KEY`, and the route's own comment says the key "ships in
client-side JS and is visible to anyone via devtools" (`route.ts:22`). The brain agrees —
"**It is not access control** — never cite it as authorization for anything."

## Priority and preconditions

**P1, not P0, and the distinction is a precondition rather than a mitigation.**
`langsmithApiKey` is `process.env.LANGSMITH_API_KEY` at module scope; with the variable unset
there is no secret to leak and the finding degrades to open proxying. There is no deployment
surface in this repo (no CI, no Dockerfile, no host config), so today this is latent. **Treat
it as P0 the moment `LANGSMITH_API_KEY` is set in any environment reachable from a network** —
at that point it is unauthenticated remote exfiltration of a production credential.

## Why an allowlist, not a stronger deny-list

Deny-listing cannot work here: the set of hosts that are *not* your LangGraph deployment is
the whole internet. The server already knows the legitimate target — `LGC_DEPLOYMENT_URL`
(`route.ts:102`) is the fallback it uses when no parameter is supplied. Compare the parsed
hostname to that one and 400 otherwise; the DNS-resolution work stays useful as
defence-in-depth for the configured host. Keep every existing check — this row adds a
condition, it does not replace the one already earned.

Two smaller notes on the same path, worth taking in the same sitting:
- `page.tsx:24` interpolates the value into the URL without `encodeURIComponent`, so a `&`
  or `#` in it injects or truncates query parameters.
- Even with the key unset, an allowlist stops the route being usable as an open request relay.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/api/copilotkit/route.ts:92](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L92) — `lgcDeploymentUrl`
[2] [src/app/api/copilotkit/route.ts:19](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L19) — `LANGSMITH_API_KEY`
[3] [src/app/api/copilotkit/route.ts:122](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L122) — `deploymentUrl`
[4] [src/app/api/copilotkit/route.ts:123](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L123) — `langsmithApiKey`
[5] [src/app/api/copilotkit/route.ts:81](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L81) — `addresses.every`
[6] [src/app/api/copilotkit/route.ts:22](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L22) — `visible to anyone via devtools`
[7] [src/lib/model-selector-provider.tsx:40](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/model-selector-provider.tsx#L40) — `lgcDeploymentUrl`
[8] [src/app/page.tsx:24](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/page.tsx#L24) — `lgcDeploymentUrl`

<!-- okf:citations:end -->
