---
schemaVersion: 1
open: 12
debt_score: 42
by_priority: { P0: 0, P1: 6, P2: 3, P3: 3 }
type: "Improvement Ledger"
title: "Improvement ledger"
description: "12 open · debt 42 (up from 25) · 0 P0 · 6 P1 · 3 P2 · 3 P3 · 5 closed"
timestamp: 2026-07-30T08:30:00.000Z
---
# Improvement ledger — research-canvas

Pass of **2026-07-30**, run against the prism-validated brain (round 3, PASS · 89/100).
17 rows total: **12 open, 5 fixed**. Ids are stable — every pre-existing row kept its id,
and no dev triage was overwritten.

## Debt trend

| Pass | Rows | Open | P0 | P1 | P2 | P3 | Debt |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 2026-07-27 | 14 | 9 | 0 | 3 | 2 | 4 | 25 |
| 2026-07-30 | 17 | 12 | 0 | 6 | 3 | 3 | **42** |

Weights: P0 = 8 · P1 = 5 · P2 = 3 · P3 = 1, summed over `status: open` rows.

**The number went up, and that is the honest reading of this pass — measurement improved,
the code did not get worse.** Nothing regressed; not one previously-closed row reopened.
The +17 decomposes as: two genuinely new P1s that no prior pass had looked for
(`lgc-deployment-url-key-exfiltration`, `python-resource-text-uncapped`, +10), one new P2
(`parallel-tool-calls-openai-only`, +3), and one escalation where the prediction came true
(`stale-model-pins` P3 → P1, +4). Expect this number to fall on the next pass without any
new work being found: four of the six open P1s are single-sitting fixes.

## Open, by leverage

Ordered by impact × ease — the top three are all high-impact, low-effort.

| Row | P | Category | Effort | Why now |
| --- | --- | --- | --- | --- |
| [stale-model-pins](improvements/stale-model-pins.md) | P1 | ops | ~10 min | The `anthropic` option 404s **today**; the pinned snapshot was retired 2025-10-28 |
| [lgc-deployment-url-key-exfiltration](improvements/lgc-deployment-url-key-exfiltration.md) | P1 | security | ~15 min | Any caller can point the proxy at their own host and receive `LANGSMITH_API_KEY` |
| [python-resource-text-uncapped](improvements/python-resource-text-uncapped.md) | P1 | performance | ~20 min | Unrecoverable 429 on one long article; the cap exists in the TS port already |
| [download-error-sentinel-blocks-retry](improvements/download-error-sentinel-blocks-retry.md) | P1 | correctness | ~10 min | `"ERROR"` is truthy, so a transient blip poisons a URL for the process lifetime |
| [next-transitive-postcss-sharp-cves](improvements/next-transitive-postcss-sharp-cves.md) | P1 | security | ~15 min | 3 high advisories fixable by `pnpm.overrides` now, without the 15 → 16 major |
| [next-dependency-cves](improvements/next-dependency-cves.md) | P1 | security | ~half day | 21 advisories on `next@15.5.15`; only the 16.x line clears them |
| [copilotkit-runtime-transitive-cves](improvements/copilotkit-runtime-transitive-cves.md) | P2 | security | ~15 min | A lockfile refresh inside the existing caret range may clear two of three |
| [fact-check-node-unguarded-toolcall](improvements/fact-check-node-unguarded-toolcall.md) | P2 | correctness | ~10 min | Four unguarded `tool_calls[0]` sites; the only node breaking the repo's own convention |
| [parallel-tool-calls-openai-only](improvements/parallel-tool-calls-openai-only.md) | P2 | correctness | ~15 min | Same file as the row above — one sitting settles both |
| [body-font-override-negates-geist](improvements/body-font-override-negates-geist.md) | P3 | product | ~5 min | Two Geist fonts loaded, then overridden with Arial |
| [resource-cache-unbounded](improvements/resource-cache-unbounded.md) | P3 | performance | ~15 min | `_RESOURCE_CACHE` never evicts |
| [submit-message-settimeout-race](improvements/submit-message-settimeout-race.md) | P3 | correctness | ~10 min | A hardcoded 30 ms sleep sequencing a state flush |

## Closed (tombstones — never removed, per contract §2/§12.4)

| Row | P | Closed | Evidence |
| --- | --- | --- | --- |
| [search-node-unguarded-toolcall](improvements/search-node-unguarded-toolcall.md) | P1 | 2026-07-23 | Guards at `search.py:67`, `:139`; cites re-pointed at the fix this pass |
| [lgc-deployment-url-ssrf-asymmetry](improvements/lgc-deployment-url-ssrf-asymmetry.md) | P2 | 2026-07-26 | `isSafeDeploymentUrl` resolves DNS and rejects every private address |
| [default-nextjs-metadata](improvements/default-nextjs-metadata.md) | P2 | 2026-07-26 | `layout.tsx:18-19` carries the real product title |
| [coagent-initialstate-divergence](improvements/coagent-initialstate-divergence.md) | P2 | 2026-07-30 | Shared `createInitialAgentState` factory (`types.ts:32`), both hooks |
| [frontend-state-any-typing](improvements/frontend-state-any-typing.md) | P2 | 2026-07-30 | `resources: Resource[]` / `logs: Log[]` (`types.ts:23-24`) |

The last two were already marked `fixed` but carried no `fixed:` date and no closure line —
and `frontend-state-any-typing` still cited `types.ts:11 :: any[]`, a token that no longer
exists at a line now belonging to an unrelated interface. Both are now real tombstones with
citations pointing at the fix.

## Security profile

`rafa audit --json` at `0c96b3c` — **the engine, not an LLM impression.**

| Tier | Ran | Tool | Findings |
| --- | --- | --- | --- |
| dependency | yes | osv-api + pnpm-audit (`pnpm-lock.yaml`) | 28 (0 critical · 13 high · 12 moderate · 3 low) |
| secrets | yes | built-in ruleset | 0 |
| sast | **no** | — | semgrep not installed |

All 28 map onto the three existing CVE rows with no new row required, and the mechanical
priority map (critical→P0 · high→P1 · moderate→P2 · low→P3) is already reflected:

| Package | Findings | Row |
| --- | --- | --- |
| `next@15.5.15` | 10 high · 9 moderate · 2 low | `next-dependency-cves` (P1) |
| `postcss@8.4.31` · `sharp@0.34.5` | 3 high · 1 moderate | `next-transitive-postcss-sharp-cves` (P1) |
| `uuid@10.0.0` · `@hono/node-server@1.19.15` · `@ai-sdk/provider-utils@3.0.30` | 2 moderate · 1 low | `copilotkit-runtime-transitive-cves` (P2) |

`lgc-deployment-url-key-exfiltration` is the one security row the audit did **not** produce —
it is an authz/data-flow finding from the LLM pass over the brain's trust map, which is
observational by definition and is not presented as scanner output.

## Coverage of this pass

Every improvement declares `blast_radius` — the row's only hub edge in the knowledge graph.
Across 17 rows the domains touched are: `agent-python` (5), `security` (4), `agent-bridge` (5),
`external-integrations` (3), `state` (2), `components` (2), `data-persistence` (2),
`routing-app-shell` (2), `build-tooling` (2), `agent-typescript` (1). `toolbox` carries no
improvement — the only mapped domain with none.

## Notes for the next pass

- **`verify-citations` skips `status: fixed` rows.** Two tombstones had silently rotted
  cites; the gate reported all-pass while they pointed at a blank line and at an unrelated
  interface. Re-checking closed rows is currently a manual step.
- **`sast: ran:false`** is a real gap in the profile, not a clean result. Installing semgrep
  would turn the third tier on; it is a machine-local prerequisite, so it is surfaced live
  rather than banked here.
- No `P0` exists, so the blast-radius-scoping exception (contract §12.5) does not fire this
  pass — every open row will surface only when work enters its region.
