---
schemaVersion: 1
verdict: PASS
round: 3
score: 89
gates: { fidelity: pass, coverage: pass }
counts: { blockers: 0, majors: 2, minors: 9 }
type: "Health Report"
title: "Brain health checklist"
description: "prism verdict PASS · score 89/100 · 0 blockers · 2 majors · 9 minors"
timestamp: 2026-07-30T08:05:00.000Z
---
# Validation report card — research-canvas · round 3

**Verdict: PASS** · score **89/100** · hard gates **pass** · **0 blockers**, 2 majors,
9 minors. The bar (score ≥ 85 · 0 blockers · majors ≤ 2 · no unflagged 5b exemplar) is met
at the major ceiling, not comfortably above it.

Round 2's blocker is **repaired and propagated** — I confirmed `8` on all three surfaces
independently (`repo-toolbox.md:7,20`, `manifest.json` as parsed, `rules/index.md:14`) and
re-counted `.agents/skills/*/SKILL.md` from `git ls-files` myself: 8 directories, 7 versions
in `rafa.json`, `dev-loop` the difference — exactly as the note now explains.

The two majors are both **new this round**, found by my own probes, not carried forward.
One is the same field-drift class one note over; the other is a genuine missing contract that
the brain currently points at the wrong file for.

Scan mode: **REFRESH**. Continuity checked mechanically against the brain repo's own HEAD and
clean.

## Machinery (procedure steps 0–2)

| check | method | result |
| --- | --- | --- |
| `@rafinery/cli` identity | live · `npm view @rafinery/cli version dist.tarball repository.url` → `0.17.0` · `git+https://github.com/rafinery-ai/rafinery.git` · tarball on the npm registry; matches the `rafa.json → rafa.cli` pin | real package, not an npx placeholder |
| `doctor` | live · `npx -y @rafinery/cli@0.17.0 doctor` | **exit 0** — 7 sensors wired, 3 hook scripts parse, heartbeat landed, blueprint coherent |
| `verify-citations` | live · same binary | **exit 0** — resolution 192/192 · completeness 34/34 · policy 5/5 · absence 18/18 · inventory 13/13 · 0 warns · 0 dangling links |
| `verify-citations --selftest` | live | **exit 0** · `checker self-test (v2): PASS` — 6/6 mutation cases (good cite passes / bad cite fails · present token flagged / absent token passes · exact inventory passes / drift flagged). The verifier is proven working, not merely re-run |
| `compile` | live | **exit 0** — 18 notes · 14 improvements · coverage 11 domains → `.rafa/manifest.json` |
| `okf check` | live | **exit 0** — 43 files · 0 errors · 2 warnings (both documented/cosmetic: `active.md` producer exemption, `log.md` non-bare-ISO heading) |
| `citation-check.json` freshness | read after my own run | `checkerVersion: 2` · `pass: true` · `at: 2026-07-30T07:25:06Z` · gate counts identical to my run — no stale record riding a push |

## Criteria checklist

`checked-by` · atlas (self-report, read from `coverage.md`) · prism (independent) · evidence.

| # | checked-by | atlas | prism | evidence (independent) |
| --- | --- | --- | --- | --- |
| A1 | prism | PASS | ✓ | No `pnpm-workspace.yaml` / `turbo.json` / `nx.json` / `lerna.json` in `git ls-files`. Three units — root `package.json`, `agents/typescript/package.json`, `agents/python/pyproject.toml` — all listed. The inert `nx` stub is real (`package.json:43`) |
| A2 | prism | PASS | ✓ | 11 domains with explicit status at `coverage.md:3` + the body table |
| A3 | prism | PASS | ✓ | Re-counted from the note files, not the table: agent-bridge 7 · routing-app-shell 2 · nine others 1 each = 18. Every mapped domain has ≥1 substantive note; no token stubs |
| A4 | prism | PASS | ✓ | Vacuous — no thin/stubbed/empty rows exist |
| A5 | checker | PASS | ✓ | `compile` exit 0. I also enumerated every `blast_radius[]` across `.rafa/improve/improvements/*.md` — 9 distinct domain names, all resolving to the declared enum |
| B1 | checker | PASS | ✓ | My own run: **exit 0**, 192/192. `coverage.md:25-27` references `citation-check.md` rather than pasting it — correct per SOP |
| B2 | prism | PASS | ✓ | **Adversarial re-grep, my own commands:** `research_agent` 18 hits / 18 cited · `DeleteResources` 9/9 · `MAX_TOTAL_RESOURCE_CHARS` 2/2 · `_RESOURCE_CACHE` 5/5 = 34, matching the checker exactly. No omitted site. Policy 5/5 — 3 token anchors on contracts, 2 justified `anchor: none`, 1 extra token anchor on a convention |
| B3 | prism | PASS | ✓ | All 18 tokens re-grepped by a second method across **all** tracked files, then split code-ext vs markdown. 14 are zero everywhere. The four non-zero — `poetry` / `remoteEndpoints` / `langGraphPlatformEndpoint` (`readme.md`), `workspace:*` (`.agents/skills/dev-loop/SKILL.md` prose), `Depends` (`.claude` skill prose) — are **markdown only**, and every note declaring them scopes its prose to "nowhere in code". `ANTHROPIC_API_KEY` is 0 across every tracked file, so `env-and-integrations`' stronger "not in code, not in docs" wording holds. Round 1's false `rafa-distill` cite is genuinely gone |
| B4 | prism | PASS | ✓ | Cross-process shape verified by reading all three: `src/lib/types.ts:19-26` (6 fields) ↔ `agents/python/src/lib/state.py:41-52` (6) ↔ `agents/typescript/src/state.ts:19-26` (5, no `citations`). The asymmetry is documented as a recorded decision, not guessed |
| B5 | prism | PASS | ✓ | Provider nesting verified against `src/app/page.tsx:13,21,28,36` and the throw at `src/lib/model-selector-provider.tsx:66` |
| C1 | prism | PASS | **partial** | **7/8**, not 8/8 — see the load-bearing test. The "how do I add W" arm of the feature task fails on one step (major J-2) |
| C2 | prism | PASS | ✓ | All 18 notes answer ≥1 work-time question; none is pure description |
| C3 | prism | PASS | ✓ | All 18 carry `type` + `domain` + ≥1 cite; all 5 contracts carry `failure:` |
| C3b | prism | PASS | ✓ | **Recomputed from the files by parsing frontmatter, not read from the report:** 66 edges across 18 notes · 0 dangling · 0 notes without an outbound link · 0 without an inbound one. Body `(/brain/...)` links all resolve; no literal `[[wikilinks]]` |
| C4 | prism | PASS | **✗** | Traversable overall, but three prose pointers misroute (m1, m4) and one section cite over-attributes the graph's branch declaration to `agent.py:18-31`, where it does not exist (part of J-2) |
| D1 | prism | PASS | ✓ | `rules/` + `playbooks/` + `coverage.md` (plus generated `index.md`, `citation-check.*`). **No `graph.json`** |
| D2 | checker | PASS | **✗** | `compile` exit 0 validates §2 frontmatter *shape*. Schema-valid is not the same as intact — `design-system-convention`'s `summary` is silently truncated by YAML plain-scalar comment parsing (major J-1) |
| D3 | checker | PASS | ✓ | `compile` exit 0, `.rafa/manifest.json` written (generated, not hand-edited) |
| §12.4 CONT | prism | (not self-checked) | ✓ | **Mechanical, against the brain repo's own HEAD** (`.rafa` is a git repo at `4d87f3e`): HEAD carries exactly 17 notes; all 17 are `M` — modified in place — **0 deleted, 0 renamed**. Exactly one new id, `state-persistence-convention`, and it is not a twin: 3 of its 7 cited files overlap `langgraph-agent-convention` (43%, under the 50% bar) and no existing title is near-equal. **No parallel brain.** `brain/coverage.md` is genuinely untracked at HEAD, independently confirming `coverage.md:29-36`'s headline claim |
| 5b | prism | (not self-checked) | ✓ | **No unflagged salient-but-wrong exemplar.** I probed eight salient conventions; every trap is flagged: `readme.md` end to end (wrong dirs, poetry, wrong `.env` paths, the omitted `NEXT_PUBLIC_COPILOTKIT_API_KEY`, the `research_agentt` typo), `vercel.json`, `dockerize.sh`, the commented `OpenAIAdapter` at `route.ts:16-17`, the hardcoded `#6766FC`/`#0E103D`, the Arial override at `globals.css:6`, `agents/typescript` as a lagging port, and the strongest one — `fact_check_node` named as the anti-exemplar the how-to tells you *not* to copy (`fact_check.py:65,114,129,141` unguarded, `:126` guarding only the forced response — all four verified in source). **The 5b override does not fire.** The silent-convention dimension is also covered: "no `"use client"` ≠ Server Component" is stated in two notes, and I verified the underlying count (2 of the 7 feature components carry the directive) |

## Load-bearing test (step 5) — 7/8

**Feature — "add a `SummariseResources` tool that writes a `summary` state field and renders it in the canvas."**
Flow → `research-chat-flow` ✓. Blast radius → `agent-state-shape-contract` (a new state key must
exist in the frontend type *and* the Python TypedDict; the TS port lags by decision) +
`resource-text-dominates-the-system-prompt` (the prompt is already the dominant cost term) ✓.
Where/convention → `langgraph-agent-convention` (the no-op `@tool` idiom, `chat.py:16-38`,
`bind_tools` at `:82`) ✓. **How do I add W → partial ✗.** `add-agent-tool-howto` is precise on the
tool schema, the bind list, the routing branch, the `emit_intermediate_state` mapping and the
`useCopilotAction` mirror — and then sends you to `agent.py` for the graph edges. For a node
reached from `chat_node` that is the wrong file (J-2). **3/4.**

**Bug — "the Fact Check panel never appears."**
Flow → `research-chat-flow` steps 3+5, including the `state.citations && length > 0` guard, which
I verified verbatim at `ResearchCanvas.tsx:202` ✓. Blast radius → `agent-state-shape-contract`
(`citations` is Python + frontend only) and `agent-typescript-parity` (on the TS backend the panel
silently never renders — confirmed: `state.ts:19-26` has five fields, `chat.ts:95`'s bind list stops
at `DeleteResources`) ✓. Where → `langgraph-agent-convention` + the `fact_check.py` cites ✓.
How-to/fix → `state-persistence-convention` supplies the non-obvious cause (a resource whose
download failed once holds a sticky `"ERROR"` at `download.py:66,81` and is skipped forever by
`chat.py:72` and `fact_check.py:81`) ✓. **4/4.**

## Scorecard

| Weight | Dimension | Score | Basis |
| --- | --- | --- | --- |
| 35 | Load-bearing | **31** | 7/8 questions answerable from the notes alone (0.875 × 35) |
| 25 | Coverage balance | **22** | All 11 domains substantive, none a stub — but 9 rest on a single note and agent-bridge holds 7 of 18. The concentration is defensible and `coverage.md`'s "Honest limits" declares it; docked for thinness, not tunneling |
| 25 | Net-positive density | **23** | 18/18 clear the "a competent dev couldn't derive this from a single file" bar, applied literally — every note is a cross-file contract, a flow, a scar or a *why*. Docked because on one note the non-obvious content never reaches the consumer: the surfaced one-liner is amputated (J-1) |
| 15 | Connectivity | **13** | 66 edges, 0 dangling, 0 orphans in either direction, no literal wikilinks. Docked for three misrouting prose pointers |
| **100** | **Composite** | **89** | |

## Findings

### BLOCKER

None. Round 2's B-1 is repaired and verified on all three surfaces.

### MAJOR

**J-1 · `design-system-convention`'s `summary` is silently truncated by YAML — the platform serves half a sentence.**

`.rafa/brain/rules/design-system-convention.md:7` is an **unquoted** YAML plain scalar containing
a `#`:

```yaml
summary: UI is shadcn "new-york" Radix primitives in components/ui with CSS-variable tokens and the cn() merge helper; feature components layer on top but hardcode the #6766FC/#0E103D brand colors inline rather than as tokens
```

In a plain scalar, ` #` opens a comment. Everything from `#6766FC` on is discarded at parse time.
What the platform actually receives (read out of `.rafa/manifest.json`, not inferred):

> `"UI is shadcn \"new-york\" Radix primitives in components/ui with CSS-variable tokens and the cn() merge helper; feature components layer on top but hardcode the"`

The sentence stops mid-clause, and the amputated half **is the note's headline value** — that the
brand accents bypass the design-token system, which is the one thing a cold agent restyling this
app needs to know. It has already propagated twice: `design-system-convention.md:18`'s
`description:` was generated from the truncated parse (it is quoted, and equally short), and
`rules/index.md:9` renders the stub to every reader of the index.

Scope: I dumped `title` + `summary` for **all 18 notes** as parsed from the manifest — this is the
**only** note affected. It is also **pre-existing**, not introduced this round (`git show
HEAD:brain/rules/design-system-convention.md` carries the identical pair), which is why no gate
caught it and why the round-2 sweep of this exact class missed it.

**Fix:** quote the scalar — `summary: "UI is shadcn 'new-york' … hardcode the #6766FC/#0E103D brand
colors inline rather than as tokens"` — then re-run `rafa okf` so `description:`, `manifest.json`
and `rules/index.md` re-materialize from the intact value. Do not hand-edit the generated surfaces.

---

**J-2 · The graph's branch declaration is a load-bearing type annotation that no note mentions — and two notes point at the wrong file for it.**

`agents/python/src/lib/chat.py:43`:

```python
) -> Command[Literal["search_node", "chat_node", "delete_node", "fact_check_node", "__end__"]]:
```

Ground truth, my own grep of `agents/python/src/`: `agent.py` declares **only** `set_entry_point`
(`:27`) and five static `add_edge` calls (`:28-32`). There is **no `add_conditional_edges`
anywhere in the Python agent**, and none of the five static edges leaves `chat_node`. So
`chat_node`'s four possible destinations are declared in exactly one place in this repository —
the `Literal` in its return-type annotation — and the annotation's members match the `goto`
targets at `chat.py:122,136,152-162` one for one, `__end__` included.

That makes it a **silent convention nothing in this repo catches**: there is no mypy, no test
suite, no CI. A Python type annotation is not enforced at runtime by any tool here, so a wrong one
fails only when the graph is exercised.

The brain does not carry it, and actively misdirects twice:
- `.rafa/brain/playbooks/add-agent-tool-howto.md:27` — "Add a graph node + edges in `agent.py` if
  it needs its own step." Correct for the node and its *outbound* edge (`agent.py:24,31` for
  `fact_check_node`); wrong for the *inbound* edge from `chat_node`, which is never in `agent.py`.
- `.rafa/brain/rules/langgraph-agent-convention.md:35-38` — "**Graph shape** (`agent.py:18-31`):
  … `chat_node` branches (to `search_node`, `delete_node`, `fact_check_node`, back to itself, or
  END)". The described branching is factually correct; the cite attributes it to a range that does
  not contain it.

A dev or cold agent following the how-to exactly would add the tool, the bind entry, the `goto`
branch, the node and the outbound edge — and leave `chat.py:43` untouched, so `chat_node` has no
declared route to the new node.

*Verification honesty:* the code facts above are all **live greps and reads**. The consequence
("the new node is unreachable") is **reasoned, not executed** — I did not run the Python graph
(no `uv` environment provisioned in this sandbox). The reasoning is grounded rather than assumed:
the annotation enumerates exactly the `goto` set including `__end__`, it was extended when
`fact_check_node` was added, and no other edge declaration for `chat_node` exists in the repo.

Contrast worth capturing in the same note: the TS agent declares the identical routing
*explicitly*, via `addConditionalEdges("chat_node", route, [...])` at
`agents/typescript/src/agent.ts:23-28`. The two backends declare chat routing in **different
files**, so the how-to's "edges in `agent.py`" is right for TS and wrong for Python.

**Fix:** add the `Command[Literal[...]]` declaration to `langgraph-agent-convention` as a named
gotcha (cite `chat.py:43` and contrast `agent.ts:23`), correct the `agent.py:18-31` section cite to
name both files, and make it step 3.5 of `add-agent-tool-howto` — "extend the `Literal` in
`chat_node`'s return annotation, or the branch you just wrote has no edge." Consider declaring
`anchor: Command[Literal` so the sites stay exhaustively cited.

*Considered and rejected as a blocker:* nothing either note states about behaviour is false — the
flow they describe matches the code. The defect is an omission plus an imprecise location, which
the SOP scores as missing essence (step 6), not a wrong claim (step 7).

### MINOR

**m1 · `repo-toolbox.md:76` under-lists the git-side hooks.** It names `post-commit`, `pre-push`,
`post-checkout`, `post-rewrite`. `git ls-files .claude/rafa/hooks` also tracks **`pre-commit`**
(plus `brain-commit.mjs`, `brain-map.mjs`, `brain-switch.mjs`, `statusline.mjs`).
Fix: add `pre-commit`, or reword to "including". *(Open since round 2.)*

**m2 · `coverage.md:225-226` states a grep that does not return zero.** "no tracked path matches
`test|spec|__tests__|.github/workflows|jest|vitest|playwright|cypress|conftest`." Running exactly
that returns `.agents/skills/tdd/tests.md`. The *conclusion* — no app test surface — is correct,
and I confirmed it three other ways: root `package.json` has no `test` script,
`agents/typescript/package.json` has one that `exit 1`s by design, and
`agents/python/pyproject.toml` declares no runner. Only the stated proof is overstated.
Fix: exclude `.agents/` and `.claude/` from the pattern as written. *(Open since round 2.)*

**m3 · `state-persistence-convention.md:50` says "the *only* store" while a second one exists.**
`_RESOURCE_CACHE` is called "the *only* store of fetched resource content", but the TS agent holds
its own — `RESOURCE_CACHE` at `agents/typescript/src/download.ts:12`, read at `:20`, written at
`:47` and `:51` with the same `"ERROR"` sentinel. The note explicitly covers the TS agent elsewhere
(its "no checkpointer at all" section), so the scope reads repo-wide. The `_RESOURCE_CACHE`
anchor's exhaustiveness is unaffected — 5/5, re-verified. Fix: scope the sentence to the Python
agent and add a line for the TS twin, which shares the sticky-`"ERROR"` trap the note already
flags. *(Open since round 2.)*

**m4 · `nextjs-app-shell-convention.md:41` points path-alias readers at the wrong file.**
"`@/` → `src/` (see `components.json` aliases …)". `components.json:13-19` lists shadcn-CLI aliases
— *consumers* of `@/`. The mapping the compiler and bundler resolve is `tsconfig.json:20-21`
(`"@/*": ["./src/*"]`), which no note cites. Note also `components.json:18` declares
`"hooks": "@/hooks"` for a directory that does not exist, so the file is a poor source of truth
here. Fix: cite `tsconfig.json:20-21` as the definition. *(Open since round 2.)*

**m5 · the refresh's one new note never states why no existing note covers it.** `rafa-scan`
step 0 requires each genuinely-new id to name that in its **body**.
`state-persistence-convention` does not; the justification lives only in `coverage.md`. Continuity
is otherwise clean (see the §12.4 row), so this is procedural. Fix: one sentence in the body.
*(Open since round 2.)*

**m6 · `build-tooling-convention`'s upstream-leftover list omits `wfcms-data.json`.** The note
enumerates `vercel.json`, `readme.md` and `dockerize.sh` (`:111`, `:116`) as stale-upstream
artifacts. Root-level `wfcms-data.json` is the same class — no tracked file references the token
(`git grep -l wfcms` outside the filename returns nothing) and its `live_demo` points at the
upstream `examples-coagents-research-canvas-ui.vercel.app` deployment. Fix: add it to the list.
*(Open since round 2.)*

**m7 · `repo-toolbox.md:33-34` carries an undeclared absence claim** (step 3b). "there is **no**
`rafa-distill` skill — distillation rides the reconciler, not a card." Truth depends entirely on a
token not existing, and this repo has already been burned twice by exactly `rafa-distill` (round 1
repaired one note listing it as real and another citing a file inside it). The `claude-skills`
inventory row (13, re-derived every run) gates the realistic drift path — adding the skill flips
the count and fails — which is why this is minor rather than major. Fix: add
`absent: rafa-distill` for exactness, so the negative claim is gated directly rather than by
arithmetic.

**m8 · `design-system-convention.md:38` carries an ungated existence claim.** "**Only two of the
seven declare `"use client"`**" — I verified it and it is **true** (`ResearchCanvas.tsx:1`,
`ModelSelector.tsx:1`; the other five carry no directive). But nothing re-checks it: the
`components` inventory row gates the count 7, not the split. It also cannot ride the `absent:`
lane, since `"use client"` *is* present in four files. This underpins the note's whole
"don't infer RSC-ness" argument. Fix: no clean mechanization — either accept it as a judgment claim
or pin it via a new inventory-style glob-with-content row (see the ratchet).

**m9 · a real cause of "Fact Check never appears" is not in the brain.** `fact_check.py:62` returns
early on an empty/whitespace `report`, resolving the tool call with "There is no report to
fact-check yet." and never calling the model — so with no draft written, the panel simply never
populates. `research-chat-flow` step 3's `FactCheckReport` bullet does not mention it. Borderline
by the non-obvious test (a single-file read shows it), which is why it is minor. Fix: one clause in
`research-chat-flow` step 3.

## Health notes

- `doctor` **exit 0** — the capture chain is alive: 7 sensors wired, `session-start.mjs` /
  `post-tool.mjs` / `user-prompt-submit.mjs` all parse, the heartbeat landed on the platform,
  vendored `.claude/` coherent, both queues empty. `gitleaks` and `semgrep` are absent but are
  optional enhancers, not gaps — `rafa audit`'s core engine (OSV dep CVEs + secrets ruleset) is
  built in and the lockfile is present.
- `.rafa/improve/improvements/lgc-deployment-url-ssrf-asymmetry.md` is a correct **tombstone**
  (`status: fixed`). Its cites are the record of what it once claimed, not live claims — explicitly
  **not** a finding.
- The `agents/typescript` drift list is knowledge, not rot. I confirmed every entry independently:
  5 nodes vs Python's 6 (`agent.ts:16-20`), no `FactCheckReport` in the `bindTools` list
  (`chat.ts:94-95`), no `citations` in `state.ts:19-26`, `MemorySaver` imported at `agent.ts:8` and
  never passed to `workflow.compile` at `:33`, and `gpt-4o` at `model.ts:21` against Python's
  `gpt-4o-mini` at `model.py:26`.
- Everything round 3's repairs claimed checks out at the code level: `search.ts:157` is the real
  guard (`:148-150` are comments, as the round-2 finding said), the "per-node table" promise is
  gone from `add-agent-tool-howto`, and `repo-toolbox`'s counts (13 / 5 / 8) are each re-derived
  correct from `git ls-files`.
- Deep claim-truth sampling this round went well beyond the SOP's ~10: I read
  `route.ts`, `agent.py`, `main.py`, `chat.py`, `fact_check.py`, `download.py`, `search.py`,
  `model.py`, `state.py`, `delete.py`, `types.ts`, `page.tsx`, `Main.tsx`, `ResearchCanvas.tsx`,
  `model-selector-provider.tsx`, `ModelSelector.tsx`, `layout.tsx`, `globals.css`,
  `components.json`, `tsconfig.json` and the TS agent's `agent.ts`/`chat.ts`/`state.ts`/`model.ts`/
  `search.ts`/`download.ts`. Every line-number claim I checked landed, with the single exception
  recorded in J-2 (a section cite, not a `cites:` entry — which is why the checker passes).

## Ratchet recommendation

**Mechanize J-1's class — it is the third consecutive round of a machine-read frontmatter field
diverging from the note's own content, and this instance is purely deterministic.** Add a
`verify-citations` lane that re-parses each note's frontmatter and **fails** when a `summary:` or
`description:` scalar is *plain* (unquoted) and its raw line contains ` #`, ` :` or a trailing `\`
— i.e. any construct that silently changes the parsed value. A stronger form: assert
`parsed(summary)` is a prefix-complete sentence by comparing its length to the raw line's length
after the key, and warn on any shortfall. That converts "the platform is served half a sentence"
from something prism has to notice by reading into an exit-1 the producer fixes before hand-off.

Round 2's recommendation (bare integer in `summary`/`description` vs a differently-counted
`inventory:` glob row) is still worth building and would additionally cover m8.
