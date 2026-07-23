---
schemaVersion: 1
id: lgc-deployment-url-ssrf-asymmetry
priority: P2
lens: security
status: open
title: The lgcDeploymentUrl SSRF guard checks host strings only — no DNS resolution, unlike the Python side
summary: isSafeDeploymentUrl blocks private hostnames by literal/regex pattern but never resolves DNS, so a public hostname pointing at a private/metadata IP passes and the proxy forwards langsmithApiKey to it — the Python download guard does resolve DNS
fix: Resolve the hostname and reject when any resolved address is private/loopback/link-local, mirroring download.py's _is_safe_url (~15 min)
leverage: { impact: medium, effort: medium }
blast_radius: [agent-bridge, external-integrations]
cites:
  - src/app/api/copilotkit/route.ts:29 :: isSafeDeploymentUrl
  - src/app/api/copilotkit/route.ts:38 :: hostname
  - agents/python/src/lib/download.py:41 :: getaddrinfo
found: 2026-07-20
---
`isSafeDeploymentUrl` (`route.ts:29-51`) is the guard that keeps the runtime proxy — which
carries `langsmithApiKey` — from being redirected at internal hosts
([copilotkit-runtime-route-convention](/brain/rules/copilotkit-runtime-route-convention.md)). It
rejects private targets purely by inspecting the URL's `hostname` string (`route.ts:38-48`:
`localhost`, `127.`, `10.`, `169.254.`, …). It never resolves DNS, so a public hostname
(`evil.example`) whose A record points at `169.254.169.254` or an internal `10.x` address passes
the check and the server then forwards the LangSmith key to that address.

The Python resource downloader guards the same SSRF class **correctly** — `_is_safe_url`
resolves via `socket.getaddrinfo` (`download.py:41`) and rejects if *any* resolved IP is
private/loopback/link-local. This is an asymmetry: the frontend proxy's guard is weaker than the
backend's for the identical threat. It compiles and passes the happy path (real LangGraph Cloud
URLs work), so nothing flags the gap. Defense-in-depth on an opt-in path (`?lgcDeploymentUrl=`),
hence P2 — but the fix is to reuse the Python side's already-correct pattern.
