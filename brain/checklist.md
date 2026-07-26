---
schemaVersion: 1
verdict: PASS
round: 3
score: 93
gates: { fidelity: pass, coverage: pass }
counts: { blockers: 0, majors: 2, minors: 4 }
type: "Health Report"
title: "Brain health checklist"
description: "prism verdict PASS · score 93/100 · 0 blockers · 2 majors · 4 minors"
timestamp: 2026-07-26T22:44:42.840Z
---

# prism — scan validation, round 3

**Repo** `/Users/rohikgalli/Desktop/research-canvas` · code sha `f83aec9` (clean tree)
**Brain** 16 notes (12 `rules/` + 4 `playbooks/`) + `coverage.md` · 13 domains

Judged fresh against current source. Prior rounds' reports were not read as evidence before
the verdict was formed; every gate below was re-run by hand and ~50 cites were
re-verified by direct file read (not by trusting the checker's table).

**Verdict: PASS** — hard gates pass, score 93 ≥ 85, 0 blockers, majors = 2 (≤ 2), and the
5b probe found **no unflagged salient-but-wrong exemplar** (the override does not fire).
Two majors remain open and are recorded below for the next pass; neither is a truth defect.

---

## Checklist

| # | criterion | checked-by | atlas (self) | prism (verify) | evidence |
|---|---|---|---|---|---|
| A1 | every code unit covered | prism | ✓ | ✓ | 3 units on disk (`src/`, `agents/python/`, `agents/typescript/`); all three carry ≥1 note. No workspace manifest exists — `git ls-files \| grep -E 'pnpm-workspace\|turbo.json\|nx.json'` → empty |
| A2 | domains enumerated with status | prism | ✓ | ✓ | 13 domains in `coverage.md:3` frontmatter, 10 `mapped` / 3 `empty`; `rafa compile` re-derives "coverage 13 domains" |
| A3 | every mapped domain has ≥1 note | prism | ✓ | ✓ | domain table `coverage.md:45-59`; each named note exists on disk |
| A4 | every empty domain states why | prism | ✓ | ~ | all three state why; `auth` is gate-backed, `data-persistence` + `testing` rest on one-time greps — see **M1** |
| B1 | citations resolve | checker | ✓ | ✓ | `npx -y @rafinery/cli@0.12.0 verify-citations` → **exit 0**, resolution **183/183** |
| B2 | contract anchors complete | checker + prism | ✓ | ✓ | completeness **28/28**, 3 anchors. Independently re-grepped: `git grep -n research_agent` / `DeleteResources` / `workspace:*` — every code hit is cited (markdown excluded per the brain's stated docs rule) |
| policy | every contract declares `anchor:` | checker | ✓ | ✓ | **4/4**; `agent-state-shape-contract` + `provider-nesting-contract` declare `anchor: none` with a stated reason |
| B3 | declared absences re-grepped | checker | ✓ | ✓ | **13/13** pass; checker WARN lane **0** |
| B3b | *undeclared* absence remainder | prism | ✓ ("none remain") | ✗ | 4 ungated existence-dependent claims found — see **M1**. `coverage.md:113`'s "No un-declared absence-shaped claim remains" is false |
| inventory | declared counts vs `git ls-files` | checker | ✓ | ✓ | **10/10**; re-derived `agents/python/src/lib/*.py` = 8, `src/components/*.tsx` = 7, `.claude/skills/*/SKILL.md` = 13 by hand |
| B4 | cross-process state shape captured | prism | ✓ | ✓ | `types.ts:19-26` (6 fields) ↔ `state.py:41-52` (6) ↔ `state.ts:19-26` (5, no `citations`) — the declared lag is real and correctly attributed |
| B5 | composition/ordering invariant captured | prism | ✓ | ✓ | `page.tsx:13→21→28→36`; throw at `model-selector-provider.tsx:66` confirmed |
| C1 | load-bearing on real work | prism | ✓ | ✓ | 5 fresh probes, 20 questions, **17 answerable** (scorecard below) |
| C2 | no note is mere code description | prism | ✓ | ✓ | 16/16 clear the non-obvious test (cross-file contract, flow, scar, or *why*) |
| C3 | frontmatter completeness | checker + prism | ✓ | ✓ | all 16 carry `type`/`domain`/≥1 cite; all 4 contracts carry `failure:` (2 `silent`, 1 `silent`, 1 `loud`) |
| C4 | cross-links traversable | prism | ✓ | ✓ | link graph 16 nodes: **0 orphans** (every note has ≥1 inbound `links:`), **0 dangling** (checker Links lane 0), **0** literal `[[…]]` in prose |
| 5b | salient-but-wrong exemplars flagged | prism | ✓ | ✓ | 9 salience paths probed; all traps flagged — see the 5b section |
| D1 | output shape | prism | ✓ | ✓ | `rules/` 12 + `playbooks/` 4 + `coverage.md`; no `graph.json` |
| D2 | frontmatter valid per contract §2 | checker | ✓ | ✓ | `rafa compile` → "16 notes · health ok" |
| D3 | compile clean | prism | ✓ | ✓ | `npx -y @rafinery/cli@0.12.0 compile` → **exit 0**, writes `.rafa/manifest.json` |
| — | checker not lying (2a) | prism | — | ✓ | ~50 cites re-verified by direct file read; **0 mismatches** |
| — | checker logic alive (2b) | prism | — | ✓ | `verify-citations --selftest` → 6/6 sub-tests, "checker self-test (v2): PASS", **exit 0** |
| — | `citation-check.json` freshness | prism | — | ✓ | `checkerVersion: 2 · pass: true · at 2026-07-26T20:59:34Z` matches the run I just made, gate-for-gate |

---

## Scorecard

| weight | dimension | measure | score |
|---|---|---|---|
| 35 | Load-bearing | 17 / 20 probe questions answerable | **30** |
| 25 | Coverage balance | 3/3 units, 10/10 mapped domains, no tunneling; 2 thinnesses declared + justified | **24** |
| 25 | Net-positive density | 16 / 16 notes non-obvious (not derivable from one file) | **25** |
| 15 | Connectivity | 0 orphans · 0 dangling · 0 literal links; one cite-region imprecision | **14** |
| | **composite** | | **93** |

### Load-bearing probes (fresh — not the ones `coverage.md` self-reports)

| probe | flow? | breaks? | where? | how to add? |
|---|---|---|---|---|
| **F1** add a `RegenerateReport` tool streaming a new `summary` field | ✓ | ✓ | ✓ | ✓ |
| **F2** require a real login before anyone can use the chat | ✓ | ✓ | ✓ | ✓ |
| **B1** deployed to Vercel: install fails, then chat never responds | ✓ | ✓ | ✓ | ✓ |
| **B2** run the TypeScript agent instead of Python | ✓ | ✓ | ✓ | ✓ |
| **B3** the Progress list resets halfway through a search | ✗ | ✗ | ✓ | ✗ |

F2 is the strongest result: `security-posture` gives the answer a cold agent would otherwise
miss — adding Next.js auth alone is useless, because `main.py:44` binds `0.0.0.0` with no
guard and anything that reaches :8000 drives the graph directly. B3 is **M2** below.

### 5b — salient-but-wrong exemplar probe (no override fired)

| salience path | most salient example | flagged as non-exemplar? |
|---|---|---|
| how to write a graph node | `fact_check_node` (newest, richest) | ✓ twice, with a per-node guard table, axis-scoped |
| how to run / deploy | `readme.md` | ✓ wrong end to end, 6 enumerated failures |
| how to deploy on Vercel | `vercel.json` | ✓ + "don't re-add `workspace:*` to make it match" |
| how to containerize | `dockerize.sh` | ✓ upstream-relative path |
| how to configure the runtime | commented-out `OpenAIAdapter` (`route.ts:16-17`) | ✓ "don't pattern-match the comment as active" |
| how to set env vars | `readme.md`'s 4-key env block | ✓ demoted, `env-and-integrations` made authoritative |
| how to style | hardcoded `#6766FC` / `#0E103D`; Arial body override | ✓ both |
| how to port to TS | the whole `agents/typescript/` tree | ✓ incl. the gpt-4o vs gpt-4o-**mini** drift |
| **silent server/client boundary** | 5 of 7 feature components have no `"use client"` | ✓ "no directive does NOT mean Server Component here" |

The last row is the SOP's first-class silent-convention dimension and the brain handles it
correctly: following the brain prevents the RSC breach. Verified against code —
`grep -n "use client" src/components/*.tsx` returns exactly `ModelSelector.tsx:1` and
`ResearchCanvas.tsx:1`.

---

## Findings

### Blockers — 0

None. Claim truth is strong: every code claim I sampled matched the source exactly,
including all four `fact_check.py` unguarded sites (`:65`, `:114`, `:129`, `:141`) against
its single response-only guard (`:126`), the `"ERROR"`-sentinel chain
(`download.py:24,66,81,97` → `chat.py:72` → `fact_check.py:81`), the `search_node`
`ToolMessage`-on-early-return detail (`search.py:145`), and all six `readme.md` failure
claims.

### Majors — 2

**M1 · Four existence-dependent claims remain ungated, and `coverage.md` asserts none do.**
Round 3 correctly hardened the `auth: empty` row into 7 `absent:` tokens — and left the
adjacent rows of the same table resting on a one-time grep. Every one of these is *true
today* (I re-grepped each); the defect is **durability**, which is exactly the 2026-06-08
ratchet's class. Sites:

| # | claim | location | proposed gate |
|---|---|---|---|
| a | "`pnpm-workspace.yaml` / `turbo.json` are absent" — A1's scoping premise that the 3 listed units are the whole tree | `coverage.md:32-34`, `:67-69` | inventory row `workspace-config :: {pnpm-workspace.yaml,turbo.json,nx.json} :: 0` |
| b | "`git ls-files` matches no `*.test.*`, `*.spec.*`, `__tests__/`, `test_*.py` or `conftest.py`" | `coverage.md:59` (`testing: empty`) | inventory rows for those globs, declared 0 |
| c | "No ORM/client of any kind in code … `aiosqlite` never imported" | `coverage.md:58` (`data-persistence: empty`) | `absent: aiosqlite` + `absent: import sqlite3` on `langgraph-agent-convention` |
| d | "`OPENAI_API_KEY` is the only one literal in **source**, and only commented-out" | `env-and-integrations:512-514` | `anchor: OPENAI_API_KEY` — the same promote-to-anchor move already used for `workspace:*` |

Compounding it: `coverage.md:113` states **"No un-declared absence-shaped claim remains;
checker WARN lane is empty."** The second clause is true (WARN = 0, confirmed); the first is
false. A self-assessment that overstates the brain's own durability is the one place a
future validator is most likely to take atlas's word for it. *Fix:* add the four gates,
then either restate that line accurately or delete it.
*Not a blocker* — no dev acts on it and no claim about the code is wrong; it misleads about
process, not about the repo.

**M2 · Missing essence: the `logs` field's lifecycle is uncaptured, and it sits in the
add-a-node path the brain's own how-to walks.** `logs` is one of the six load-bearing state
fields, and it is the only one with four writers and two truncation points — none of which
any note mentions:

- `src/app/Main.tsx:49` clears it on every chat submit (followed by a 30 ms `setTimeout`
  the brain also doesn't mention).
- **`agents/python/src/lib/search.py:134` does `state["logs"] = []` — a full wipe
  mid-node**, after the per-query steps complete and before the "No resources extracted"
  append at `:140`. This is why the Progress list visibly resets partway through a search.
- `download.py:93,109`, `search.py:76,96` and `fact_check.py:71,121` all capture
  `logs_offset = len(state["logs"])` and then write back **positionally**
  (`state["logs"][logs_offset + i]["done"] = True`). Append a log between the offset capture
  and the indexed write and you mark the wrong step done.

`agent-state-shape-contract` gives `logs` one line ("drives the `Progress` step list");
`research-chat-flow` step 4 says only that nodes "push progress with
`copilotkit_emit_state`". `add-agent-tool-howto` step 3 tells a dev to "add a graph node",
and every existing node uses the offset idiom — so the trap is directly on the documented
path. Note also that the brain designates `search_node` as the node to copy; a dev copying
it wholesale inherits the `:134` log wipe. *(This does **not** fire 5b: the brain scopes
"copy `search_node`'s shape" to the guarding axis explicitly, the same axis-scoping it
already applies to `fact_check_node`.)*
*Fix:* a `logs` lifecycle paragraph in `agent-state-shape-contract` (writers, the two clear
points, the positional-offset rule), cross-linked from `add-agent-tool-howto` step 3 and
from `research-chat-flow` step 4.

### Minors — 4

**m1 · Cite region points at the comment, not the guard.**
`agent-typescript-parity:182` cites the mirrored TS guards as `search.ts:53`,
`search.ts:148-150`. `:53` is guard 1 (`if (!aiMessage.tool_calls?.length)`); `:148-150` is
the *comment block* describing guard 2 — the guard itself is `agents/typescript/src/search.ts:157`.
Prose-only (not a machine-checked cite), so the checker can't catch it. Use `:157`, or
`:148-157`.

**m2 · The commit that ships this brain describes a different brain.**
`f83aec9` reads "founding scan — 28 notes, 15 domains, prism PASS (93)" with a body claiming
"18-item improve ledger" and a "1.32x" benchmark. The artifact is a **refresh** with **16
notes / 13 domains**, and this is round 3 of a repair loop. `.rafa/` is gitignored
(`.gitignore:39`), so the commit is empty and exists only as the mirror-hook checkpoint —
which makes its message the entire shipped record. Outside `.rafa/brain/`, so out of gate
scope, but the conductor should amend it before push.

**m3 · A count-shaped convention claim with no mechanical gate.**
`design-system-convention:435` — "**Only two of the seven** declare `"use client"`". True
today (verified), and it is the premise of the silent-RSC-boundary guidance, which is the
brain's highest-value convention. It should be re-counted, not remembered: an inventory row
over `src/components/*.tsx` containing `use client` would carry it. Same class as M1 but
lower stakes, since drift in the likely direction (someone *adds* the directive) is benign.

**m4 · A stated fact whose *why* was left in the code.**
`agent-state-shape-contract:97-98` records that `createInitialAgentState` is "the single
seed used by both `useCoAgent` call sites" but not the failure it prevents. `types.ts:28-31`
spells it out — "Both `useCoAgent` hooks (Main.tsx, ResearchCanvas.tsx) must seed the SAME
shape or the shared coagent state becomes mount-order dependent." Two hooks sharing one
coagent by name is exactly the cross-file, non-obvious *why* the brain exists to carry;
right now the code comment is smarter than the note.

---

## Health notes

`npx -y @rafinery/cli@0.12.0 doctor` → **exit 0**, "all clear". Provisioning resolves
(`gallirohik-research-canvas`), all 7 git/session sensors wired, all 3 `.mjs` hook scripts
parse, both queues empty, 7/7 skill dependencies verified, and the platform round-trip
heartbeat landed. Capture is alive — a distill run today would have something to collect.
Two optional security enhancers are absent (`gitleaks`, `semgrep`); the core engine (OSV dep
CVEs + secrets ruleset) is built in, so this is a nice-to-have, not a gap.

## The ratchet — judgment findings that should become machine checks

1. **M1(a)–(d) and m3 are all one mechanizable class.** The checker already has both lanes
   (`absent:` tokens, `inventory:` globs). What it lacks is a heuristic over **`coverage.md`
   itself**: an `empty`-status domain row whose prose contains an absence phrase
   ("no X", "nothing is", "matches no", "never imported", "are absent") and whose domain has
   **no** backing `absent:` token or `inventory:` row → WARN. That single rule would have
   caught (a), (b) and (c) mechanically, and it generalises the round-3 `auth` fix instead of
   leaving it as a one-off.
2. **A `git ls-files`-backed content-inventory lane** (glob + required-substring + expected
   count) would mechanize m3 and any future "N of M files do/don't contain token T" claim —
   the shape that keeps recurring in silent-convention notes.
3. **Prose cite regions.** m1 is the second imprecise prose `file:line` this brain has
   shipped. A lint that resolves `` `path:NN` `` patterns **in note bodies** (not just the
   `cites:` block) and warns when the line is inside a comment would push it into the machine.
