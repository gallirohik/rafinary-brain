---
schemaVersion: 1
id: lgc-allowlist
plan: lgc-allowlist
parent: null
kind: epic
title: Allowlist the LangGraph deployment host to close the LANGSMITH_API_KEY exfiltration gap
description: >-
  isSafeDeploymentUrl (route.ts:59-84) only denies private/internal targets, so any
  public host a caller names in ?lgcDeploymentUrl= still passes and receives
  langsmithApiKey — open P1 lgc-deployment-url-key-exfiltration. This is the
  residual the DNS-resolution SSRF fix (lgc-deployment-url-ssrf-asymmetry, P2,
  already resolved) did not close: deny-private is not allow-known.
approach: allowlist the configured deployment host (LGC_DEPLOYMENT_URL) alongside the existing DNS/private-IP checks, never replacing them; encode the param client-side
domains: [agent-bridge, external-integrations, security]
branch: fix/lgc-deployment-url-key-exfiltration
type: "Plan Epic"
tags: [lgc-allowlist, epic]
timestamp: 2026-08-01T07:43:05.616Z
---

## Context

`lgcDeploymentUrl` originates in the browser's own query string
(`model-selector-provider.tsx:40` → `page.tsx:24`), so it is fully
caller-controlled. The route reads it (`route.ts:92`) and runs
`isSafeDeploymentUrl`, whose terminal test is `addresses.every(addr =>
!isPrivate…)` (`route.ts:81`) — a deny-private check, not an allow-known one.
The value that passes becomes `deploymentUrl` on `LangGraphAgent` alongside
`langsmithApiKey` (`route.ts:122-123`), so the LangSmith key rides the
outbound request to whatever public host was named. The only thing in front
of this is `isAuthorized`, documented in-repo as not access control
(`route.ts:22-23`).

P1 not P0: `LANGSMITH_API_KEY` is unset today and there is no deployment
surface in this repo, so the finding is latent. It becomes P0 the moment the
key is set in any network-reachable environment.

The server already knows its legitimate target — `LGC_DEPLOYMENT_URL`
(`route.ts:102`) is the fallback used when no `?lgcDeploymentUrl=` param is
supplied. Comparing the parsed hostname to that one closes the gap; deny-list
hardening (DNS resolution, private-IP checks) stays as defence-in-depth.

Bundled in the same sitting: `unencoded-lgc-deployment-url` (open) —
`page.tsx:24` interpolates `lgcDeploymentUrl` into the query string without
`encodeURIComponent`, so a `&`/`#` in the value injects or truncates query
parameters. Same file, same path, named in the P1 row's own fix text.

Ledger: [lgc-deployment-url-key-exfiltration](https://github.com/gallirohik/research-canvas) (P1, security).

## Done-check (rollup)

The child task's Done-check passes: the allowlist guard is added alongside
the existing checks (nothing removed/weakened), the client-side value is
encoded, and typecheck is clean.

## Log

- 2026-08-01 (conductor, close-out): Child task `lgc-allowlist-fix` prism-validated
  PASS at build time (validation_tier full) — the exfiltration path is empirically
  closed (a public host clearing every pre-existing check is now rejected on
  hostname mismatch alone) and the encodeURIComponent fix verified by round-trip.
  Epic flipped to `status: done`; active pointer cleared. Ledger rows
  lgc-deployment-url-key-exfiltration and unencoded-lgc-deployment-url reported
  fixed.
- 2026-08-01 (conductor): Plan-lite authored by hand after the driven `rafa run plan`
  ladder's audit step hard-blocked on 28 pre-existing, out-of-scope dependency CVEs
  (`rafa audit` exits 1 on any finding by design — unrelated to this fix; already
  tracked as next-dependency-cves / next-transitive-postcss-sharp-cves /
  copilotkit-runtime-transitive-cves). Recall, bloom's blast-radius pull, and the
  security-audit transparency pass were completed via the ladder before it was
  abandoned; this plan carries that grounding forward.
