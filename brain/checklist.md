---
schemaVersion: 1
verdict: PASS
round: 1
score: 91
gates: { fidelity: pass, coverage: pass }
counts: { blockers: 0, majors: 2, minors: 3 }
---
# prism report card — founding scan of `@copilotkit-examples/research-canvas`

Independent validation. Every check re-run against the code; nothing in `coverage.md`
trusted on its word. Hard gates pass, both flagged low-confidence claims verified TRUE,
no unflagged salient-but-wrong exemplar. Two majors are content-accuracy defects in
otherwise-strong notes — fixable, non-blocking; they do not misroute the core work-time
questions.

## Machinery (step 0–2)
| Check | Result | Evidence |
|---|---|---|
| `verify-citations` | exit 0 · 118/118 res · 27/27 compl · 4/4 policy · 0/0 absence · 6/6 inv | re-run myself; matches `citation-check.json` (v2, pass, at 2026-07-20T11:42:07Z — not stale) |
| `verify-citations --selftest` | PASS · exit 0 (6/6 mutation cases) | verifier proven to fail bad cites + pass good |
| `compile` | 15 notes · 11 domains → manifest.json | re-run |
| `doctor` | all clear — sensors wired, scripts parse, heartbeat lands | capture chain alive |
| trust-but-verify cites (hand-read ~15) | all match | langgraph.json, main.py, model.py, provider.tsx, settings.json, .mcp.json, types.ts, state.py, page.tsx, Main.tsx, ResearchCanvas.tsx, ts/model.ts, chat.py:47/52 |
| adversarial completeness grep | site-list == grep-hits | `research_agent` 18/18, `DeleteResources` 9/9 (code-only) |

## Acceptance criteria (checked-by · atlas self · prism verify · evidence)
| # | checked-by | atlas | prism | evidence |
|---|---|---|---|---|
| A1 apps/pkgs listed | prism | ✓ | ✓ | web + agents/python + agents/typescript + toolbox all in coverage; monorepo leaf noted |
| A2 every domain status | prism | ✓ | ✓ | 11 domains, all `mapped`/`stubbed`, none unlisted |
| A3 mapped→≥1 note | prism | ✓ | ✓ | 9 mapped domains each ≥1 substantive note (not stubs) |
| A4 stubbed states why | prism | ✓ | ✓ | data-persistence + auth both cited-justified |
| B1 checker exit 0 | checker | ✓ | ✓ | re-run exit 0; `citation-check.json` current |
| B2 contract anchors | checker | ✓ | ✓ | 4/4 policy; both token anchors complete by independent grep |
| B3 absence grep-proven | prism | ✓ | ✗ | **M2** — env note asserts provider-key names "not literal in source" with NO `absent:` declared; undeclared existence-dependent claim (2026-06-08 stale class) |
| B4 cross-process shape | prism | ✓ | ✓ | `agent-state-shape-contract`: 5-field set × 3 files verified (types.ts/state.py/state.ts) |
| B5 composition/ordering | prism | ✓ | ✓ | `provider-nesting-contract` (anchor: none) verified against page.tsx nesting |
| C1 route feature+bug | prism | ✓ | ✓ | add-tool feature + "canvas won't update" bug — all 4 questions answerable, no blind search |
| C2 every note earns place | prism | ✓ | ✓ | all 15 answer ≥1 work-time question |
| C3 type/domain/cite/failure | checker | ✓ | ✓ | compile passes; contracts carry `failure:` |
| C4 contracts+flows link | prism | ✓ | ✓ | dense `links:` graph; 0 orphans, 0 dangling |
| D1 rules+playbooks+coverage | prism | ✓ | ✓ | no graph.json |
| D2 frontmatter §2 | checker | ✓ | ✓ | compile exit 0 |
| D3 compile exit 0 | checker | ✓ | ✓ | manifest.json written |

## Flagged low-confidence claims — adversarial verdict
1. **model-selection-flow (both graph ids → same graph; model rides `state.model`)** — **CONFIRMED TRUE.**
   Both `langgraph.json` files map `research_agent` AND `research_agent_google_genai` to the
   same `./src/agent.py:graph` (agents/python/langgraph.json:6-7). `main.py:20,27` register both
   FastAPI endpoints with the *same* `graph=graph` object. `get_model` switches purely on
   `state.get("model","openai")` (model.py:18-47) — agent name never enters model choice.
   Frontend derives name = `research_agent`, only `google_genai` flips it
   (model-selector-provider.tsx:42-45). All four dropdown values resolve to a real model; none
   is "dead." The note's reasoning holds and is genuinely non-obvious. High-value.
2. **repo-toolbox citations into `.claude/**` / `.mcp.json`** — **files PRESENT in working tree**
   (untracked on `chore/rafa-init`, present = fine). `.mcp.json:3` rafinery, `:7` RAFA_MCP_KEY,
   `.claude/settings.json:4` Read(.rafa, `:5` @rafinery/cli, `.claude/commands/rafa.md:2` version 2.0.2
   — all verified by direct read. Hooks/statusline claims accurate. BUT the *skills enumeration*
   in prose is wrong — see M1.

## Scorecard
| Dimension | Weight | Score | Note |
|---|---|---|---|
| Load-bearing | 35 | 35 | feature + bug both fully routed; silent server/client + HITL + name contracts all captured |
| Coverage balance | 25 | 24 | full breadth, no tunneling; both agents + toolbox + stubs justified |
| Net-positive density | 25 | 21 | almost every note is a cross-file contract/flow/why; docked for M1/M2 accuracy defects |
| Connectivity | 15 | 11 | 0 orphans, 0 dangling (checker); docked for 2 link text/target mismatches |
| **Composite** | 100 | **91** | |

## Findings ledger

### Major
- **M1 — `rules/repo-toolbox.md:20-24` · wrong skills enumeration.** Prose lists "Skills (10):
  `rafa`, rafa-scan, rafa-plan, rafa-build, rafa-improve, rafa-validate, rafa-distill,
  rafa-insights, rafa-leverage, rafa-okf." Ground truth (`ls .claude/skills/`) is 10 dirs:
  rafa-build, rafa-distill, rafa-improve, rafa-insights, rafa-leverage, rafa-okf, rafa-plan,
  **rafa-sage**, rafa-scan, rafa-validate. The note invents a nonexistent `rafa` skill (there is
  no `.claude/skills/rafa/`; `rafa` is the `/rafa` *command*, not a skill) and **omits the real
  `rafa-sage`** (confirmed present, `name: rafa-sage`). Count is coincidentally still 10, masking
  the swap. An index note whose job is accurate enumeration is factually wrong in exactly that
  dimension. *Fix:* replace `rafa` with `rafa-sage` in the skills list; describe `/rafa` under
  Command only. (The agent-cards list of 5 is correct.)
- **M2 — `rules/env-and-integrations.md:26-28` · undeclared absence claim + imprecise wording.**
  Claims `OPENAI_API_KEY`/`ANTHROPIC_API_KEY`/`XAI_API_KEY` "names are **not literal in this
  repo's source**." This is an existence-dependent claim with **no `absent:` declared**, so the
  checker's B3 gate never re-greps it — the exact class that went stale in the 2026-06-08 ratchet.
  Also imprecise: `OPENAI_API_KEY` *does* appear literally at `src/app/api/copilotkit/route.ts:14`
  (a commented-out `OpenAIAdapter` line). `ANTHROPIC_API_KEY`/`XAI_API_KEY` are truly absent (grep
  clean). The substance ("read implicitly by the LangChain constructors") is correct and useful.
  *Fix:* declare `absent: ANTHROPIC_API_KEY` + `absent: XAI_API_KEY` in frontmatter so the gate
  re-greps forever; reword the OPENAI line to note it appears only commented-out (route.ts:14) /
  state the code-vs-comment exclusion.

### Minor
- **m1 — `rules/build-tooling-convention.md` body** references "`concurrently` (`package.json:6`)"
  but `concurrently` is at `package.json:9`; line 6 is `"build": "next build"`. The *cite*
  (package.json:9) is correct — only the stray in-prose line number is off. *Fix:* drop or correct
  the `:6`.
- **m2 — `rules/agent-name-contract.md` body** link `[model-selector-provider](/brain/rules/copilotkit-runtime-route-convention.md)`
  — anchor text names a code file / different note than the target. Resolves (not dangling), but
  misleads a reader. *Fix:* point text and target at the same note (or `model-selection-flow`).
- **m3 — `playbooks/model-selection-flow.md` body** link `[langgraph.json](/brain/rules/agent-name-contract.md)`
  — same text/target mismatch pattern (text says a filename, target is a note). Resolves. *Fix:*
  align text with target.

## Health notes
- Capture machinery green (`doctor` all clear) — not a brain blocker; recorded for the report card.

## Verdict
**PASS** · score 91 · 0 blockers · 2 majors (≤2) · 3 minors · no unflagged salient-but-wrong
exemplar (5b clear). Hard gates (fidelity + coverage) pass; both low-confidence claims verified.
M1 and M2 are cheap, high-signal corrections worth folding in before push — recommend the
conductor have atlas fix both (and optionally the minors) on the next pass, but they do not gate.

## Ratchet recommendation
M2 is the recurring class the B3 gate exists for. Consider a checker heuristic that flags an
`_API_KEY`/`_TOKEN`-shaped token named in prose with a negation ("not", "no", "never") and no
`absent:` declared — pushing the undeclared-absence catch down into the machine.
