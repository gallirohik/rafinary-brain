---
schemaVersion: 1
domains: { agent-bridge: mapped, agent-python: mapped, agent-typescript: mapped, build-tooling: mapped, components: mapped, data-persistence: mapped, external-integrations: mapped, routing-app-shell: mapped, security: mapped, state: mapped, toolbox: mapped }
inventory:
  - route-pages :: src/app/**/page.tsx :: 1
  - layouts :: src/app/**/layout.tsx :: 1
  - api-routes :: src/app/api/**/route.ts :: 1
  - components :: src/components/*.tsx :: 7
  - ui-primitives :: src/components/ui/*.tsx :: 6
  - py-agent-modules :: agents/python/src/lib/*.py :: 8
  - ts-agent-modules :: agents/typescript/src/*.ts :: 8
  - langgraph-manifests :: agents/*/langgraph.json :: 2
  - claude-skills :: .claude/skills/*/SKILL.md :: 13
  - agent-skills :: .agents/skills/*/SKILL.md :: 8
  - agent-cards :: .claude/agents/*.md :: 5
  - env-example :: .env.example :: 0
  - middleware :: **/middleware.ts :: 0
type: "Coverage Report"
title: "Scan coverage"
description: "11 domains — 11 mapped"
timestamp: 2026-07-30T07:03:57.993Z
---
# Coverage — research-canvas

Refresh scan, 2026-07-30. Fidelity evidence is in
[citation-check.md](/brain/citation-check.md) — not pasted here, so it can never drift from
what the checker actually said.

## Why this file was missing

The org brain shipped 17 notes and 14 improvements with **no `coverage.md`**. Because the
registry lane skips when the file is absent, `rafa compile` reported `coverage —` and
`get_coverage` served `domains: []` — every `domain`, `blast_radius[]` and `domains[]` in
the brain was unenforced, and the map consumers orient on was empty. That is the single
largest finding of this refresh; authoring this file closes it and turns the domain enum
into a gate.

## Workspace (A1)

**Not a monorepo.** One root `package.json` (`@copilotkit-examples/research-canvas`, Next
15.5.15 / React 19) and one nested, de-workspaced package (`agents/typescript`, its own
`pnpm-lock.yaml`). No `pnpm-workspace.yaml`, no `turbo.json`, no `lerna.json`/`nx.json`
(the root carries only an inert `nx.targets.build.cache` stub). The Python agent
(`agents/python`) is a `uv`/`pyproject.toml` project, outside JS packaging entirely.

| Unit | Path | Toolchain | Mapped by |
| --- | --- | --- | --- |
| root app | `/` | next · npm | routing-app-shell, components, build-tooling |
| Python agent | `agents/python` | uv · FastAPI · LangGraph | agent-python |
| TypeScript agent | `agents/typescript` | pnpm · LangGraph JS | agent-typescript |

## Domains (A2 · A3 · A4)

Eleven domains, all `mapped` — every one carries ≥1 note, so there is no `thin`/`stubbed`/
`empty` row to justify. Note counts are shown because several domains rest on a single
note; that is coverage, but it is thin coverage, and worth knowing before you trust it.

| Domain | Status | Notes | Covered by |
| --- | --- | --- | --- |
| agent-bridge | mapped | 7 | agent-name-contract · copilotkit-runtime-route-convention · delete-resources-hitl-contract · resource-text-dominates-the-system-prompt · add-agent-tool-howto · model-selection-flow · research-chat-flow |
| agent-python | mapped | 1 | langgraph-agent-convention |
| agent-typescript | mapped | 1 | agent-typescript-parity |
| build-tooling | mapped | 1 | build-tooling-convention |
| components | mapped | 1 | design-system-convention |
| data-persistence | mapped | 1 | state-persistence-convention **(new this refresh)** |
| external-integrations | mapped | 1 | env-and-integrations |
| routing-app-shell | mapped | 2 | nextjs-app-shell-convention · provider-nesting-contract |
| security | mapped | 1 | security-posture |
| state | mapped | 1 | agent-state-shape-contract |
| toolbox | mapped | 1 | repo-toolbox |

`data-persistence` was the one domain referenced by improvements
(`download-error-sentinel-blocks-retry`, `resource-cache-unbounded`) with **zero** notes
behind it. It now has one.

## What changed this refresh

Ids are stable; every existing concept was updated in place. No note was retired, and no
new id was minted for a concept an existing note already covered.

**Repaired (drift against current code):**
- `delete-resources-hitl-contract` — four `agents/typescript/src/chat.ts` cites shifted +5
  (37/38/57/82 → 42/43/62/95) when the `MAX_TOTAL_RESOURCE_CHARS` cap landed. Body line
  refs updated to match.
- `research-chat-flow` — `route.ts:105 :: handleRequest` → `:135`; the route grew a
  DNS-resolving `isSafeDeploymentUrl` SSRF guard, which is now a described step in the
  flow (it 400s before proxying, a failure mode the flow previously didn't mention).
- `repo-toolbox` — listed a `rafa-distill` skill that **does not exist** (the real 13th is
  `rafa-migrate`), and claimed 7 harness-neutral skills when `git ls-files` counts 8
  (`dev-loop`). Both are prose-level drift no citation check could catch, which is why the
  counts are now pinned as `claude-skills` / `agent-skills` in `inventory:` above.
  **Round 2 caught this repair as incomplete:** the 7→8 correction reached the body but
  not the `summary:`/`description:` frontmatter, which rides `manifest.json` verbatim and
  propagates to `rules/index.md` — so the brain contradicted itself on two surfaces while
  the gate-enforced inventory row said 8. Both fields now corrected, and they name the
  7-vs-8 distinction explicitly (7 = `rafa.json` installed versions, 8 = directories)
  so the next reader doesn't "fix" it back.
- `langgraph-agent-convention`, `agent-typescript-parity` — gained links + a paragraph
  each pointing at the new persistence note; the TS parity note gained its missing
  "no checkpointer at all" drift entry.

**Added:**
- `state-persistence-convention` (`data-persistence`) — no note covered where data lives.
  It answers "is anything saved?" with a verified **no**: four `absent:` tokens
  (`SqliteSaver`, `localStorage`, `prisma`, `drizzle`) re-grepped every run, an exhaustive
  `_RESOURCE_CACHE` anchor, the declared-but-unimported sqlite checkpointer trap, and the
  TS agent's unused `MemorySaver` import.

**Repaired in the prism round (Gate 2, round 1 → ITERATE, 92/100):** prism found a class
this scan missed — **notes asserting things about themselves**, which no lane checks
because bodies are never parsed.
- `env-and-integrations` cited `.claude/skills/rafa-distill/SKILL.md:10` as where
  `ANTHROPIC_API_KEY` occurs. That skill does not exist and the token occurs in **no**
  tracked file — the exact twin of the `rafa-distill` drift repaired in `repo-toolbox`,
  missed here, leaving the brain contradicting itself. Rewritten to state the real trap:
  the key is read implicitly by the LangChain constructor, so it is greppable nowhere.
- Four claims of the form "this is re-checked every run" named guarantees that were never
  declared (`absent: poetry`, `remoteEndpoints`, `langGraphPlatformEndpoint`; inventory
  rows `env-example`, `middleware`). Every underlying fact was true; the self-description
  was not. All five now declared — absence 15→18, inventory 11→13.
- `agent-typescript-parity` put `bindTools` at `chat.ts:82`; it is at `:94-95` (the same
  +13 shift repaired next door).
- Stale rendered `okf:citations` blocks and 5 missing OKF `type` fields were fixed by
  running `rafa okf` — the maintainer re-materialize — rather than hand-editing blocks
  that carry a "do not hand-edit" header. `rafa okf check` now exits 0.

**Repaired after Gate 2 PASS (round 3 majors, fixed rather than deferred):**
- **A YAML bug was corrupting what the platform serves.** `design-system-convention`'s
  `summary:` was an unquoted plain scalar containing ` #6766FC`, so YAML parsed the rest as
  a comment: the manifest, the generated `description:`, and `rules/index.md` all carried
  the summary truncated at *"…but hardcode the"* — amputating the note's entire headline
  point. Quoted, and the stamped `description:` corrected by hand (the emitter fills that
  field only when absent, so it had frozen the truncated text). A repo-wide sweep for the
  same hazard — unquoted `title`/`summary`/`description` containing ` #`, ` : ` or a
  trailing backslash — found this as the **only** instance across all 32 notes and
  improvements.
- **`chat_node`'s routing was misattributed.** Two notes sent readers to `agent.py` for
  chat_node's branching. There is **no** `add_conditional_edges` in the Python agent at
  all: the legal destinations are declared once, in the `Command[Literal[...]]` return
  annotation at `chat.py:43`. Adding a node reachable from `chat_node` requires editing
  that annotation, and with no mypy, no tests and no CI in this repo, omitting it is a
  silent routing failure. Both `langgraph-agent-convention` and `add-agent-tool-howto` now
  state this, with the TS port's visible `addConditionalEdges` cited as the contrast.

**Considered and rejected:** a standalone `fact-check-flow` playbook. The end-to-end
fact-check path is already traced in `research-chat-flow` (steps 3–5) and
`langgraph-agent-convention`; a fourth copy would duplicate, not add, and duplication is
how a brain starts to mislead.

## Acceptance criteria

### A · Coverage (breadth)
- **A1 PASS** — every unit in workspace config is listed above; the repo is single-package
  plus two agent projects, none omitted.
- **A2 PASS** — 11 domains, each with an explicit status.
- **A3 PASS** — every `mapped` domain has ≥1 note (table above). `data-persistence` was
  the sole violation at scan start and is now closed.
- **A4 PASS** — vacuous: no `thin`/`stubbed`/`empty` rows to justify.
- **A5 PASS** — mechanical. `rafa compile` resolves every note `domain` and every
  improvement `blast_radius[]` against this file's `domains` and exits 0.

### B · Fidelity
- **B1 PASS** — `rafa verify-citations` exits 0: resolution **195/195**, completeness
  **34/34** (4 anchors), policy 5/5, absence **18/18**, inventory **13/13**, 0 warns,
  0 dangling links. Evidence in [citation-check.md](/brain/citation-check.md). Baseline at
  scan start was **9 FAILED** (5 resolution + 4 completeness), all repaired above.
  `rafa okf check` also exits 0 (bundle conforms to OKF v0.1).
- **B2 PASS** — all **5** contracts declare `anchor:`. Three carry token anchors
  (`research_agent`, `DeleteResources`, `MAX_TOTAL_RESOURCE_CHARS`); two declare
  `anchor: none` with a stated reason — `agent-state-shape-contract` (the field SET is the
  contract) and `provider-nesting-contract` (composition/ordering). A fourth token anchor,
  `_RESOURCE_CACHE`, rides the new `state-persistence-convention`: anchors are declared for
  load-bearing tokens, not only where the policy gate forces them onto contracts. Every
  token anchor is exhaustively cited — the checker asserts each code occurrence is a site.
- **B3 PASS** — **18** `absent:` tokens re-grepped and confirmed (11 pre-existing, 4 from
  the new persistence note, 3 added in the prism round below). Three notes *claimed* a
  token was gate-checked while never declaring it — the claim, not the fact, was wrong;
  declaring `poetry`, `remoteEndpoints`, `langGraphPlatformEndpoint` made the prose true
  mechanically rather than softening it.
  Contract-site lists count **code** occurrences only; docs, markdown and comments are
  excluded, applied identically to every contract. `redis` was *considered* as an absence
  token and **dropped**: it occurs in `.agents/skills/frontend-design/LICENSE.txt`, so the
  claim would not have been cleanly true.
- **B4 PASS** — cross-process/state-shape contracts captured: `agent-state-shape-contract`
  (frontend type ↔ Python TypedDict ↔ TS annotation, incl. the intentional `citations`
  asymmetry) and `resource-text-dominates-the-system-prompt` (capped in TS, uncapped in
  Python).
- **B5 PASS** — composition/ordering captured: `provider-nesting-contract`
  (ModelSelectorProvider ⊃ CopilotKit ⊃ coagent hooks).

### C · Work-time value
- **C1 PASS**, traced against two real tasks with no blind searching:
  - *Feature — "add a Summarize-resources tool that streams into the canvas":* flow →
    `research-chat-flow`; blast radius → `agent-state-shape-contract` (a new state key
    must exist in all three state files) + `delete-resources-hitl-contract` (if it needs
    UI); where/convention → `langgraph-agent-convention` (no-op `@tool` idiom,
    `chat.py:16-38`, `bind_tools` at `:82`); how-to → `add-agent-tool-howto`, which also
    warns to copy `search_node`'s `tool_calls` guard and **not** `fact_check_node`'s.
  - *Bug — "the Fact Check panel never appears":* flow → `research-chat-flow` steps 3+5
    (the `:202` guard); blast radius → `agent-state-shape-contract` (`citations` is
    Python + frontend only) and `agent-typescript-parity` (on the TS backend the panel
    silently never renders); where → `langgraph-agent-convention`; and
    `state-persistence-convention` supplies the non-obvious cause — a resource whose
    download failed once holds a sticky `"ERROR"` in `_RESOURCE_CACHE` and is invisible to
    the fact-checker for the life of the process.
- **C2 PASS** — every note answers ≥1 of the four questions; none is pure description.
- **C3 PASS** — all 18 notes carry `type`, `domain`, ≥1 cite; all 5 contracts carry
  `failure:`. This box **failed at scan start**: `agent-state-shape-contract` was missing
  the required `failure:` field entirely. It is now `silent`, which is what its own body
  already described — a field mismatch crosses the process boundary without a type error
  and simply doesn't render.
- **C3b PASS** — **66 link edges across 18 notes, 0 dangling.** Both multi-note domains are
  interlinked, not hub-only: `agent-bridge` (7 notes, **17** intra-domain edges) and
  `routing-app-shell` (2 notes, **2** edges — nextjs-app-shell-convention ↔
  provider-nesting-contract). The nine single-note domains cannot hold an intra-domain edge
  by definition; each is linked outward, and every note in the brain carries ≥1 `links:`
  entry. `state-persistence-convention` was authored with 5 outbound links and given 2
  inbound ones (from `langgraph-agent-convention` and `agent-typescript-parity`) so it is
  reachable by a graph walk, not just by lexical match.
- **C4 PASS** — contracts and flows carry bundle-relative markdown links; blast radius and
  end-to-end flow are traversable by following them.

### D · Format & contract
- **D1 PASS** — output is exactly `brain/rules/`, `brain/playbooks/`, `brain/coverage.md`.
  No `graph.json`.
- **D2 PASS** — frontmatter valid per contract §2 on all 18 notes.
- **D3 PASS** — `rafa compile` exits 0 and writes `.rafa/manifest.json` (generated; never
  hand-edited).

## Honest limits

- **Nine of eleven domains rest on a single note.** That satisfies A3 and each note is
  broad, but single-note coverage is the thinnest kind that still counts as mapped. The
  concentration is real: `agent-bridge` holds 7 of 18 notes because the CopilotKit ↔
  LangGraph seam is where this repo's silent failures actually live.
- **`agents/typescript` is mapped as an accepted-lagging port, not audited line by line.**
  Its drift list is the knowledge; if it ever becomes the live backend, it needs its own
  scan pass.
- **Lock files and `node_modules` are out of scope** for citation. Dependency-version
  claims live in the improvement ledger (CVE rows), not in notes, so they rot in one
  place with a re-runnable audit behind them.
- **No test surface exists to map.** Grep-proven, not assumed: no tracked path matches
  `test|spec|__tests__|.github/workflows|jest|vitest|playwright|cypress|conftest`, and no
  package manifest declares a test runner (`package.json`, `agents/typescript/package.json`,
  `agents/python/pyproject.toml`). So no domain covers testing, and the toolbox's
  `tdd`/`grilling` skills are unexercised here. The root `package.json` has no `test`
  script; `agents/typescript` has one that `exit 1`s by design.
