---
schemaVersion: 1
id: copilotkit-runtime-transitive-cves
priority: P2
category: security
status: open
title: Three advisories ride in under @copilotkit/runtime — uuid, @hono/node-server, @ai-sdk/provider-utils
summary: The CopilotKit runtime that backs the app's only server route pulls uuid@10.0.0 (missing buffer bounds check), @hono/node-server@1.19.15 (Windows path traversal in serve-static) and @ai-sdk/provider-utils@3.0.30 (uncontrolled resource consumption, no fix published); the caret range on @copilotkit/runtime means a lockfile refresh may clear the first two
fix: Refresh the lockfile within the existing `^1.63.2` range (`pnpm update @copilotkit/runtime`), then re-run `npx @rafinery/cli audit`; add pnpm.overrides for uuid >= 13.0.1 / @hono/node-server >= 2.0.5 only if the refresh does not move them (~15 min)
leverage: { impact: low, effort: low }
blast_radius: [agent-bridge, external-integrations, security]
cites:
  - package.json:19 :: @copilotkit/runtime
  - pnpm-lock.yaml:3669 :: uuid@10.0.0
  - pnpm-lock.yaml:438 :: '@hono/node-server@1.19.15'
  - src/app/api/copilotkit/route.ts:7 :: @copilotkit/runtime
found: 2026-07-27
---
Source: `npx @rafinery/cli audit --json` (`rafa.audit/v1`, run 2026-07-26), dependency tier.
Machine-sourced from the envelope.

All three are **transitive under `@copilotkit/runtime@1.63.2`** (`chain.direct: false`), the
package imported at `src/app/api/copilotkit/route.ts:7`. None are `dev:true`. Priority is the
mechanical map on the highest severity present: **moderate -> P2**. Grouped into one row
because they share one parent and one fix action.

| package | severity | advisory | fixedIn |
| --- | --- | --- | --- |
| uuid 10.0.0 | moderate | GHSA-w5hq-g… (CVE-2026-41907, CVE-2026-41988) — missing buffer bounds check in v3/v5/v6 when `buf` is provided | 13.0.1 |
| @hono/node-server 1.19.15 | moderate | GHSA-frvp-7… — path traversal in `serve-static` on Windows | 2.0.5 |
| @ai-sdk/provider-utils 3.0.30 | low | GHSA-866g-f… (CVE-2026-8769) — uncontrolled resource consumption | **none published** |

Chain for the last one is `@copilotkit/runtime@1.63.2 -> @ai-sdk/google-vertex@3.0.158 ->
@ai-sdk/provider-utils@3.0.30`.

## Reachability — brain-grounded annotation (priority-neutral)

- **Server-exposed via `agent-bridge`**: this is the dependency tree of the app's single
  server entry point ([security-posture](/brain/playbooks/security-posture.md),
  [copilotkit-runtime-route-convention](/brain/rules/copilotkit-runtime-route-convention.md)),
  so these run in the request path — unlike the postcss/sharp row.
- **uuid**: the advisory requires a caller passing an explicit `buf` argument to v3/v5/v6.
  That is internal CopilotKit usage; no first-party code in this repo calls `uuid` at all.
- **@hono/node-server**: the path traversal is scoped to `serve-static` **on Windows**. This
  route serves no static assets through Hono, and the deploy target is not Windows — call this
  effectively inert here.
- **@ai-sdk/provider-utils**: reached only through the Vertex provider branch. Note the app's
  own model router (`get_model`) offers openai / anthropic / google_genai / grok
  ([model-selection-flow](/brain/playbooks/model-selection-flow.md)) and the Next.js adapter is
  an `EmptyAdapter` (`route.ts:18`) — the LLM calls happen in the Python agent, not here. No
  fix is published, so this one is a watch item regardless.

None of these downgrade the mapped priority; they explain why the row sits behind the two
`next` rows in the queue despite being on the exposed path.

## Note on the fix

`@copilotkit/runtime` is declared as `"^1.63.2"` (`package.json:19`), so a plain lockfile
refresh can move the transitives without any manifest edit — try that first and let the audit
confirm the delta. Reach for `pnpm.overrides` only if the refresh leaves them pinned, and
check the runtime still boots afterwards: uuid 10 -> 13 is three majors, and forcing it under
a parent that expects v10 is exactly the kind of override that breaks at request time rather
than at install time.
