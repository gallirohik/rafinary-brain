---
schemaVersion: 1
open: 9
debt_score: 16
by_priority: { P0: 0, P1: 1, P2: 4, P3: 4 }
---
# Improvement ledger — `@copilotkit-examples/research-canvas`

First bloom pass, indexed off the founding brain (prism PASS, score 91). This is a well-built
CopilotKit example — no P0s. The findings are the **silent rot** no gate catches: unguarded
happy-path assumptions, a shipped-boilerplate defect, a security guard weaker than its twin, and
type/consistency drift across the cross-process state contract.

**Debt score: 16** (weighted open items: P1×4 + P2×2 + P3×1). Baseline — no prior run to trend against.

Dependency-CVE scan (directive 7): `npm audit` could not resolve here — this repo is a
`workspace:*` monorepo leaf ([build-tooling-convention](/brain/rules/build-tooling-convention.md)),
so CVE auditing must run from the CopilotKit monorepo root (`nx` / pnpm workspace), not this
folder in isolation. No LLM-pretend audit was substituted.

## By priority

| Priority | Count |
| --- | --- |
| P0 | 0 |
| P1 | 1 |
| P2 | 4 |
| P3 | 4 |

## By lens

| Lens | Count |
| --- | --- |
| correctness | 2 |
| security | 1 |
| architecture | 2 |
| product | 2 |
| performance | 1 |
| ops | 1 |

## Open items (leverage-ranked)

| ID | P | Lens | Leverage (impact/effort) | Title |
| --- | --- | --- | --- | --- |
| default-nextjs-metadata | P2 | product | high / low | Ships "Create Next App" title + description to prod |
| search-node-unguarded-toolcall | P1 | correctness | medium / low | search_node assumes a tool call always comes back — empty response kills the run |
| frontend-state-any-typing | P2 | architecture | medium / low | AgentState types resources/logs as `any[]` while concrete types exist |
| coagent-initialstate-divergence | P2 | architecture | medium / low | Two useCoAgent hooks seed different initialState for the same agent |
| lgc-deployment-url-ssrf-asymmetry | P2 | security | medium / medium | lgcDeploymentUrl guard does string checks only — no DNS resolution |
| body-font-override-negates-geist | P3 | product | low / low | globals.css Arial override negates the loaded Geist fonts |
| stale-model-pins | P3 | ops | low / low | Anthropic/Google model IDs pinned to aging snapshots |
| submit-message-settimeout-race | P3 | correctness | low / low | Chat submit uses a hardcoded 30ms delay to sequence state |
| resource-cache-unbounded | P3 | performance | low / medium | `_RESOURCE_CACHE` grows unbounded for the process lifetime |

## Lead with (10-minute fixes, take them while you're in the file)

1. **default-nextjs-metadata** (P2, high/low) — `layout.tsx:18-19` still says "Create Next App".
   Two-minute fix, user-visible on every deploy.
2. **search-node-unguarded-toolcall** (P1, medium/low) — the one real risk in the primary
   backend: guard `tool_calls[0]` in `search.py` so a provider that skips the forced tool call
   doesn't abort the run.
3. **frontend-state-any-typing** (P2, medium/low) — `types.ts:11-12` → `Resource[]` + a `Log`
   type; restores compile-time safety on the state contract for free.
