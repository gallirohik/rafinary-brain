# Refresh log — gallirohik-research-canvas

## Correction context (2026-07-27)
A prior session ran a founding scan without pulling the existing org brain (15 notes + 9 improvements from PRs #11-13 reconciled on `main`), producing 28 duplicate-named notes. That PR (#14) was closed unmerged; `origin/main` was untouched. This session restored the real brain content and is running a proper refresh.

## Round 1 — 2026-07-27
- **atlas refresh**: read all 15 existing notes + 9 improvements. 4 had wrong claims (build-tooling-convention's stale monorepo claim, design-system-convention's inverted "all use client" claim, agent-typescript-parity's stale node-count claim, copilotkit-runtime-route-convention's stale sync-check claim), rest had citation drift only (line numbers moved). Added 1 new note (`security-posture.md`, genuinely uncovered domain) + authored `coverage.md` for the first time (didn't exist in the older brain schema). Note count 15 → 16. verify-citations exit 0, compile exit 0.
- **conductor gate 1** (independent re-run): verify-citations exit 0 (153/153 · 28/28 · 4/4 · 2/2 · 8/8), compile exit 0.
- **prism gate 2**: verdict **ITERATE** · score 89 · 0 blockers · 4 majors · 4 minors.
  - M1 (§5b, forces iterate): `langgraph-agent-convention.md` calls the `tool_calls[0]` guard "universal" and says new nodes should mirror `search_node`, but `fact_check_node` itself violates the guard at 4 sites (fact_check.py:65,114,129,141) — the newest, most-salient node is an unflagged non-exemplar.
  - M2: `security-posture.md` claims `use server`/`middleware.ts` absence is "declared below" but only `add_middleware` is actually declared in frontmatter — false durability guarantee.
  - M3: missing note on the `"ERROR"` cache-poisoning bug (download.py:66,81,97) — a real silent bug, absent from the whole brain.
  - M4: `agent-state-shape-contract.md` is `type: contract` with no `failure:` field, contradicting coverage.md's own claim that all 4 contracts have one.
  - Minors: one mislabeled link, 3 drifted line-refs, coverage.md note-count inconsistency, B1 table linked not pasted.
  - Action: spawn atlas to fix all 4 majors + minors, re-verify, re-validate.
- **atlas fix pass**: flagged `fact_check_node` as a guard-convention non-exemplar across 3 notes, declared `absent: use server` in `security-posture.md`, added the `"ERROR"`-sentinel cache-poisoning gotcha to `langgraph-agent-convention.md` (+ 2 other notes), added `failure: silent` to `agent-state-shape-contract.md`, plus 4 minor line-ref/link fixes and a rewritten `coverage.md` (note counts reconciled, B1 pasted). verify-citations exit 0 (167/167), compile exit 0.

## Round 2 — 2026-07-27
- **conductor gate 1** (independent re-run): verify-citations exit 0, compile exit 0.
- **prism gate 2**: verdict **ITERATE** · score 92 · 0 blockers · 3 majors · 3 minors. Round 1's 4 majors confirmed genuinely fixed.
  - M1: the `x-api-key` 401 lockout (`route.ts:26,87-89` — unset key 401s every request, app silently dead) is framed by all 3 notes mentioning it as "cosmetic," never as an operational trap; no `.env.example` exists.
  - M2 (§5b): `readme.md` is unflagged legacy — instructs `cd agent-py`/`poetry install`/uncommenting `remoteEndpoints` in `route.ts`, none of which exist (repo is `uv`-managed, 0 occurrences of `remoteEndpoints`); `env-and-integrations.md:48` even cites it as a positive authority.
  - M3: the "no auth/identity at all" absence claim in `security-posture.md`/`coverage.md` is grep-proven-once with no `absent:` gate.
  - Minors: 3 body links with label≠target, a debug note omitting its own `MODEL` env override, one wrong line ref.
  - Action: spawn atlas for round 3 (final allowed round) to fix M1-M3 + minors.
- **atlas fix pass**: reframed the `x-api-key` env var as a required runtime input (not just cosmetic security) across 3 notes with 401-lockout cites, flagged `readme.md` as a 3rd non-exemplar alongside `vercel.json`/`dockerize.sh` in `build-tooling-convention.md` (6 cited failures, 3 new `absent:` gates), gated the "no auth" claim with 7 `absent:` tokens in `security-posture.md`, plus 6 minor link/line-ref fixes. verify-citations exit 0 (183/183), compile exit 0.

## Round 3 — 2026-07-27 (final allowed round)
- **conductor gate 1** (independent re-run): verify-citations exit 0, compile exit 0.
- **prism gate 2**: verdict **PASS** · score 93 · 0 blockers · 2 majors · 4 minors. 5b override did not fire.
  - M1 (non-blocking): 4 more existence-dependent claims still ungated (workspace-absence premise, testing-empty grep list, `aiosqlite` never imported, `OPENAI_API_KEY` literal-only claim) — same ratchet class as before, left for a future pass.
  - M2 (non-blocking, missing essence): the `logs` array lifecycle (full wipe at `search.py:134`, positional indexing across 3 files) is undocumented, sits on the exact path `add-agent-tool-howto.md` tells readers to copy from `search_node`.
  - Minors: one cite pointing at a comment not the guard line, the empty-commit message describing a different (stale, wrong) brain shape than what's actually on disk, an ungated count claim, a `types.ts` comment not surfaced in the contract note.
- **Decision**: PASS → proceeding to Improve (bloom), merging into the 9 existing improvements without duplication.

## Improve — 2026-07-27
- **bloom**: refreshed all 9 existing items (0 status flips, all cites still resolved), added 5 new: `download-error-sentinel-blocks-retry` (P1, the "ERROR" cache-poisoning bug), `fact-check-node-unguarded-toolcall` (P2, not covered by the already-fixed `search-node-unguarded-toolcall`), 3 CVE rows (`next-dependency-cves` P1, `next-transitive-postcss-sharp-cves` P1, `copilotkit-runtime-transitive-cves` P2). Final: 14 rows, 9 open, debt score 20 (first-ever audit run — not a regression, prior debt was simply unmeasured). Security profile ran: 28 findings (0 critical/13 high/12 moderate/3 low), secrets 0 findings, sast not run (no semgrep).
- Gates: verify-citations exit 0 (183/183), compile exit 0 (16 notes · 14 improvements).
