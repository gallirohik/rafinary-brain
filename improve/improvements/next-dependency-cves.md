---
schemaVersion: 1
id: next-dependency-cves
priority: P1
category: security
status: open
title: next@15.5.15 carries 21 open advisories (10 high) — fixed only by the 16.x line
summary: The exact-pinned direct dependency next@15.5.15 has 21 OSV advisories against it (10 high / 9 moderate / 2 low), spanning middleware-bypass, SSRF in Server Actions and rewrites, cache poisoning, XSS and DoS; every one is cleared by upgrading to >= 16.2.11
fix: Bump the exact pin `"next": "15.5.15"` to >= 16.2.11 (a 15 -> 16 major; read the upgrade guide, then re-run `npx @rafinery/cli audit`) (~half a day)
leverage: { impact: high, effort: high }
blast_radius: [routing-app-shell, agent-bridge, security, build-tooling]
cites:
  - package.json:27 :: next
  - pnpm-lock.yaml:2816 :: next@15.5.15
  - src/app/api/copilotkit/route.ts:86 :: export const POST
found: 2026-07-27
type: Improvement
description: "The exact-pinned direct dependency next@15.5.15 has 21 OSV advisories against it (10 high / 9 moderate / 2 low), spanning middleware-bypass, SSRF in Server Actions and rewrites, cache poisoning, XSS and DoS; every one is cleared by upgrading to >= 16.2.11"
timestamp: 2026-07-27
tags: [security, P1]
---
Source: `npx @rafinery/cli audit --json` (`rafa.audit/v1`, run 2026-07-26), dependency tier —
tool `osv-api+pnpm-audit (pnpm-lock.yaml)`. Machine-sourced; not re-guessed.

`next` is a **direct** dependency (`chain.direct: true`, path `next@15.5.15`) pinned exactly
at `15.5.15` (`package.json:27`, resolved at `pnpm-lock.yaml:2816`). Twenty-one advisories
resolve against that version. Priority is the mechanical map on the highest severity present:
**high -> P1**. None are flagged `dev:true`.

## The advisories, by shared fix

| fixedIn | count | classes |
| --- | --- | --- |
| 16.2.5 | 11 | middleware/proxy bypass (App Router segment-prefetch, Pages Router, dynamic route params), SSRF, DoS with Server Components, connection-exhaustion DoS, RSC cache poisoning, XSS in `beforeInteractive`, Image Optimization DoS, middleware redirect cache poisoning |
| 16.2.6 | 1 | middleware/proxy bypass (App Router) |
| 16.2.11 | 8 | SSRF in Server Actions on custom servers, SSRF in rewrites, DoS in App Router Server Actions, response-body cache confusion (x2), unbounded Server Action payload on Edge, unauthenticated disclosure of internal Server Function endpoints, Image Optimization DoS via SVG |

Highest CVSS in the set is 7.5 (e.g. GHSA-267c-6grr-h53f / CVE-2026-44575, App Router
middleware bypass via segment-prefetch routes). One upgrade target — **>= 16.2.11** — clears
all twenty-one, which is why this is a single row rather than twenty-one.

## Reachability — brain-grounded annotation (priority-neutral)

Annotation only; it informs triage and never downgrades the mapped priority.

- **Server-exposed via `agent-bridge`**: the app has exactly one server-side entry point,
  `export const POST` at `src/app/api/copilotkit/route.ts:86`
  ([security-posture](/brain/playbooks/security-posture.md)). The Next.js server itself is
  internet-facing wherever this deploys, so the DoS, cache-poisoning and Server-Function
  disclosure classes apply directly.
- **Middleware-bypass class: not reachable today.** There is no `middleware.ts` anywhere in
  the repo, and the brain records `absent: add_middleware` for the security domain. These
  advisories describe *authorization checks in middleware being bypassed*; with no middleware
  and no auth boundary to bypass, they have no current attack surface here. They become live
  the moment anyone adds middleware — which is the argument for upgrading before that, not
  after.
- **Server Actions class: not reachable today.** The brain records `absent: "use server"`;
  the app uses no Server Actions.
- **Image Optimization class: not reachable today.** No `next/image` import exists in `src/`.

So the honest picture: roughly half the set is inert against *this* app's current shape, and
the rest (DoS, RSC cache poisoning, XSS in scripts, Server Function endpoint disclosure) is
live on the one exposed route. The finding stands at P1 because the mechanical map says high
-> P1 and because every "not reachable today" line above is one ordinary feature commit away
from being reachable.

## Effort note

This is the one row in the ledger that is deliberately **not** a 10-minute fix: 15 -> 16 is a
major. It is filed at `effort: high` so it is triaged as planned work rather than picked up
mid-task. If the major has to wait, the two transitive rows
(`next-transitive-postcss-sharp-cves`, `copilotkit-runtime-transitive-cves`) are the cheap
levers available in the meantime.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [package.json:27](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/package.json#L27) — `next`
[2] [pnpm-lock.yaml:2816](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/pnpm-lock.yaml#L2816) — `next@15.5.15`
[3] [src/app/api/copilotkit/route.ts:86](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/src/app/api/copilotkit/route.ts#L86) — `export const POST`

<!-- okf:citations:end -->
