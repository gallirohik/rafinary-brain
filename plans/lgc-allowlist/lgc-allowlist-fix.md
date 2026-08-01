---
schemaVersion: 1
id: lgc-allowlist-fix
plan: lgc-allowlist
parent: lgc-allowlist
kind: task
title: Allowlist LGC_DEPLOYMENT_URL's host in isSafeDeploymentUrl; encode lgcDeploymentUrl in page.tsx
description: >-
  isSafeDeploymentUrl (route.ts:59-84) is deny-private only, so
  https://attacker.example passes and receives langsmithApiKey. Add an
  allowlist check against LGC_DEPLOYMENT_URL's host alongside the existing DNS
  resolution and private-IP checks (never replacing them). While in this
  path, wrap lgcDeploymentUrl in encodeURIComponent at page.tsx:24
  (unencoded-lgc-deployment-url) so a `&`/`#` in the value can't inject or
  truncate query parameters.
approach: derive the allowlisted hostname from the same process.env.LGC_DEPLOYMENT_URL value route.ts:102 already reads for the fallback (one source of truth, not two independent env reads); compare isSafeDeploymentUrl's resolved hostname against it and reject on mismatch (and when unset/unparseable, reject every caller-supplied lgcDeploymentUrl with 400); encodeURIComponent in page.tsx's runtimeUrl template
status: done
priority: 2
estimate: 1
validation_tier: full
type: "Plan Task"
tags: [lgc-allowlist, task]
timestamp: 2026-08-01T07:43:05.616Z
---

## Done-check

- `isSafeDeploymentUrl` in `src/app/api/copilotkit/route.ts` rejects any
  `lgcDeploymentUrl` whose parsed hostname does not case-insensitively equal
  `LGC_DEPLOYMENT_URL`'s parsed hostname, in addition to (not replacing) the
  existing protocol / `localhost`/`.local` / DNS-resolution / private-IP
  checks.
- When `LGC_DEPLOYMENT_URL` is unset or fails to parse, any caller-supplied
  `lgcDeploymentUrl` is rejected with 400 (no known host to allow). Requests
  that supply **no** `lgcDeploymentUrl` param are unaffected by this change:
  `deploymentUrl` still falls back to `LGC_DEPLOYMENT_URL` (route.ts:102) and
  the runtime selection at route.ts:118 is untouched — when neither is set,
  the route continues to serve `LangGraphHttpAgent` against
  `REMOTE_ACTION_URL`/localhost:8000 (route.ts:93-118). The 400 applies only
  to caller-supplied values; rejection and fallback are mutually exclusive,
  never both claimed for the same request.
- `src/app/page.tsx`'s `runtimeUrl` template wraps `lgcDeploymentUrl` in
  `encodeURIComponent(...)` before interpolating it into the query string.
- `pnpm exec tsc --noEmit` exits 0 with no new errors (baseline is already
  clean at HEAD).
- Manual trace against the edited source: a request whose `lgcDeploymentUrl`
  hostname equals the configured `LGC_DEPLOYMENT_URL` host still passes
  (existing behavior preserved); a request naming a different public https
  host that resolves to a public, non-private address — `https://example.com`
  (verified: resolves to public IPv4/IPv6 addresses, clears the protocol /
  `.local` / DNS / private-IP checks, and is therefore accepted by the
  *pre-fix* guard) — is rejected with 400 post-fix solely because its
  hostname ≠ `LGC_DEPLOYMENT_URL`'s hostname. (`https://attacker.example` is
  NOT a valid exemplar: `.example` is RFC 2606 reserved and does not
  resolve, so it is already rejected by the existing DNS-resolution catch
  regardless of the allowlist.)

## Log

- 2026-08-01 (atlas): Implemented. `route.ts`: added module-scope `allowedDeploymentHost`
  derived from `LGC_DEPLOYMENT_URL` (null if unset/unparseable), and
  `isSafeDeploymentUrl` now rejects any hostname that doesn't match it — added
  alongside the existing protocol/localhost/DNS/private-IP checks, all untouched.
  `page.tsx`: `runtimeUrl` wraps `lgcDeploymentUrl` in `encodeURIComponent`.
  `tsc --noEmit` exits 0.
- 2026-08-01 (prism, PASS): Build-time Done-check gate, validation_tier full. Live
  byte-exact source-slice harness against real DNS proved the actual vulnerability
  closed: a different public host (example.org, resolves to public non-private
  addresses, cleared every pre-fix check) was ACCEPTed pre-fix and is REJECTed
  post-fix; the configured/allowed host still passes; unset/unparseable
  LGC_DEPLOYMENT_URL rejects everything; the no-param fallback path is
  byte-identical to HEAD (untouched). encodeURIComponent round-trip verified for
  `&`/`#` payloads. Residual (non-blocking, matches Done-check's literal
  "hostname" wording, flagged as a follow-up not this task): the allowlist
  compares hostname only, not port — a different service on another port of the
  same allowed host would still pass. Status → done.
