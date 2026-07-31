---
schemaVersion: 1
id: repo-toolbox
type: convention
domain: toolbox
title: The committed agent toolbox — rafa skills, harness-neutral .agents skills, the rafinery MCP server, and permissions
summary: "The repo ships the rafa engineering SOP as 13 .claude skills + 5 agent cards + the /rafa command, PLUS 8 harness-neutral skills under .agents/skills (Claude Code · Codex · Cursor all read these; rafa.json records 7 installed versions — dev-loop is authored here, not installed), pinned to @rafinery/cli 0.18.2 and wired to the rafinery knowledge MCP over HTTP with a bearer key from env; scan/plan/build are DRIVEN by the rafa run ladder and brain recall is query-first, graded verified > derived > authored"
links: [env-and-integrations, security-posture]
cites:
  - .claude/skills/rafa-scan/SKILL.md:2 :: rafa-scan
  - .claude/commands/rafa.md:2 :: version
  - .claude/commands/rafa.md:220 :: The driver
  - .claude/commands/rafa.md:240 :: .rafa/runs/
  - .mcp.json:3 :: rafinery
  - .mcp.json:7 :: RAFA_MCP_KEY
  - .claude/settings.json:5 :: @rafinery/cli
  - .claude/settings.json:4 :: Read(.rafa
  - .claude/settings.json:10 :: SessionStart
  - .agents/skills/tdd/SKILL.md:2 :: tdd
  - .agents/skills/vercel-composition-patterns/SKILL.md:2 :: vercel-composition-patterns
  - .claude/rafa/contract.md:642 :: The query plane
  - .claude/rafa/contract.md:617 :: query_schema
  - rafa.json:11 :: skills
  - rafa.json:8 :: 0.18.2
description: "The repo ships the rafa engineering SOP as 13 .claude skills + 5 agent cards + the /rafa command, PLUS 8 harness-neutral skills under .agents/skills (Claude Code · Codex · Cursor all read these; rafa.json records 7 installed versions — dev-loop is authored here, not installed), pinned to @rafinery/cli 0.18.2 and wired to the rafinery knowledge MCP over HTTP with a bearer key from env; scan/plan/build are DRIVEN by the rafa run ladder and brain recall is query-first, graded verified > derived > authored"
tags: [toolbox]
timestamp: 2026-07-26T22:44:42.840Z
---
There is no app-specific tooling in this repo — the committed toolbox is **rafa** (the
engineering SOP layer) plus a set of harness-neutral skill dependencies it installed. This
note is the index to both. Check here before hand-rolling a procedure.

**rafa skills** (`.claude/skills/rafa-*/SKILL.md`, **13**): `rafa-build`, `rafa-commit`,
`rafa-improve`, `rafa-insights`, `rafa-leverage`, `rafa-migrate`, `rafa-okf`, `rafa-plan`,
`rafa-review`, `rafa-sage`, `rafa-scan`, `rafa-security`, `rafa-validate`. Each is a
procedure card with `name:` == its directory (`rafa-scan/SKILL.md:2`) and a one-line
`description:`. Invoke via the Skill tool by name. (`rafa-migrate` backs BOTH
`/rafa migrate` and `/rafa update`; there is **no** `rafa-distill` skill — distillation
rides the reconciler, not a card.)

**Harness-neutral skills** (`.agents/skills/*/SKILL.md`, **8**): `dev-loop`, `tdd`,
`frontend-design`, `vercel-composition-patterns`, `requesting-code-review`,
`improve-codebase-architecture`, `grill-me`, `grilling`. Same `name:`/`description:`
frontmatter shape
(`.agents/skills/tdd/SKILL.md:2`,
`.agents/skills/vercel-composition-patterns/SKILL.md:2`). These live outside `.claude/`
**on purpose** — `.agents/` is the one convention Claude Code, Codex and Cursor all read,
so the same skill works whichever harness a teammate uses. They are consent-installed by
`rafa update` and their versions are recorded in `rafa.json`'s
`rafa.skills.installed` map (`rafa.json:11`), with a parallel `declined: []` list. Adding a
skill means installing it there AND recording the version in `rafa.json` — an
`.agents/skills/` directory with no `rafa.json` entry is un-tracked drift.

**The one standing exception:** `installed` lists **7** versions but `.agents/skills/` holds
**8** directories. The extra is `dev-loop` — rafa's own end-to-end algorithm, authored in
this repo and published to the harness-neutral path rather than consent-installed from a
registry, so it has no version to record. Don't "fix" the count by inventing a `rafa.json`
entry for it; the counts are pinned as `claude-skills` / `agent-skills` in
[coverage.md](/brain/coverage.md)'s `inventory:`, which re-derives both from `git ls-files`
on every check — so if a NINTH directory appears, the gate fails and you'll know it's real
drift rather than this exception.

**Command**: `.claude/commands/rafa.md` — the `/rafa <verb>` slash command, `version: 2.2.0`
(`:2`).

**The CLI pin** (`rafa.json`): `rafa.cli` is **`0.18.2`** (`:8`). This is the version of
`@rafinery/cli` the whole SOP assumes, and it decides which `rafa` subcommands actually
exist — the skills reference verbs (`rafa run`, `rafa okf`, `rafa hydrate`, `rafa facts`)
that a lower pin does not ship, and one of them (`rafa push`) is *retired* and refuses.
Run `npx @rafinery/cli --help` against the pin rather than trusting a remembered verb list.
The cite above is **deliberately brittle**: the token is the version string itself, so the
next bump fails `verify-citations` and forces this note back through a refresh, instead of
letting the number rot silently in prose — which is exactly how this note previously came
to list a `rafa-distill` skill that never existed.

**Driven verbs — `scan` · `plan` · `build` run on a ladder, not from memory** (`:220`,
the `## The driver` section of the command card). The CLI owns the SEQUENCE and the agent
supplies the judgment: `rafa run <verb>` opens (or resumes) a run and hands back ONE step
at a time. Steps are of two kinds — a `does` step the driver already executed with a
receipt (preflight pull, recall, hydrate, the citation gate, the capture checkpoint: read
its evidence, never re-run it), and an `asks` step which is the agent's work, closed with
`rafa run advance --note="…"`. **An out-of-order advance is a refusal, not an error to
route around**; `rafa run status` re-orients a resumed session, `rafa run rewind --to=`
undoes a wrong turn, `rafa run abandon` ends one honestly. The run record lives in
`.rafa/runs/` (`:240`) and is the receipt that each step happened, at a sha. This exists
because recall and capture were the two steps long sessions skipped by forgetting, and a
skipped recall is invisible — it looks identical to "the brain had nothing".

**Agent cards** (`.claude/agents/*.md`, 5): `atlas` (scan/build/plan), `bloom` (improve),
`prism` (validate), `sage` (learnings), `compass` (user insights). These are contract-gated
(§10) but are workforce config, not app code.

**MCP server** (`.mcp.json`): one server, `rafinery` (`:3`), `type: http`, url
`https://dev.rafinery.ai/api/mcp`, authenticated `Authorization: Bearer ${RAFA_MCP_KEY}`
(`:7`) — the env var is expanded at load; the raw key lives in gitignored
`.claude/settings.local.json` / `~/.config/rafinery/credentials.json`, never committed. This
is the read-only knowledge MCP over the ingested brain. Its surface is **two doors, not
one** — and the older "just search it" habit is now the wrong default:

- **Query-first is the standing rule** (contract §9a, `.claude/rafa/contract.md:642`).
  `search_knowledge` ranks by WORDS and is the right door only when free text is your
  *sole* anchor — the weakest rung. Whenever you already hold something exact (an open
  file, a rule id, a symbol, a domain, a plan/task id), that is a lookup, not a guess:
  `query_knowledge` answers it with predicates (`match · where · traverse · at · groupBy ·
  return · limit`). Scope is never a predicate — repo and tenant come from the key.
- **Introspect the vocabulary, never paste one.** `query_schema` (`contract.md:617`)
  serves the live node kinds, graded edge types and this repo's actual topic list,
  generated from the OKF registry. Call it once per session and compose against what it
  returned; a hardcoded enum goes stale the day a type is minted.
- **Results carry a GRADE, and the grade decides what you may do** — `verified` (a `cites`
  edge the build re-greps: act on it, navigate to `file:line`) > `derived` (a mechanical
  join: trust it as navigation, it makes no claim about the code) > `authored` (an
  asserted `links:`/`supersedes` edge nothing re-checks: follow as a lead, confirm against
  code first — the only grade that can be quietly wrong). No result at all is a **gap**:
  author the knowledge or record it, never guess past it.
- `resolve_cite` reads the CODE at a citation with no checkout, so a note's claim can be
  quoted against the real line rather than trusted.

`ask_knowledge` remains the guarded synthesis read for thin clients (a Slack bot, a
dashboard); an agent with its own model should prefer query/search + `get_*` and compose
itself.

**Permissions** (`.claude/settings.json`): pre-allows `Read(.rafa/**)` (`:4`),
`Bash(npx @rafinery/cli:*)` (`:5`), `Bash(rafa:*)` (`:6`). The file also registers three
hooks — `SessionStart` digest (`:10`, matcher `startup|resume|clear`), `PostToolUse`
dirty-sensor (`:21`, matcher `Edit|Write|MultiEdit|NotebookEdit`), `UserPromptSubmit` reflex
(`:32`) — and a rafa status line (`:44`), all pointing at `.claude/rafa/hooks/*.mjs`.
Alongside them, `.claude/rafa/hooks/` also ships **five** git-side hooks (`pre-commit`,
`post-commit`, `pre-push`, `post-checkout`, `post-rewrite`) that mirror and checkpoint the
brain; those are installed into `.git/hooks`, not declared in `settings.json`, so
`settings.json` is not the full hook inventory — list the directory. The mirror logic they
share lives in `.claude/rafa/hooks/brain-commit.mjs`; the post-commit mirror is the only
writer of the brain branch, and a session must be on a feature branch for it to fire (it
no-ops on `main`).
Provider/service keys are covered in
[env-and-integrations](/brain/rules/env-and-integrations.md).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [.claude/skills/rafa-scan/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/skills/rafa-scan/SKILL.md#L2) — `rafa-scan`
[2] [.claude/commands/rafa.md:2](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/commands/rafa.md#L2) — `version`
[3] [.claude/commands/rafa.md:220](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/commands/rafa.md#L220) — `The driver`
[4] [.claude/commands/rafa.md:240](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/commands/rafa.md#L240) — `.rafa/runs/`
[5] [.mcp.json:3](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.mcp.json#L3) — `rafinery`
[6] [.mcp.json:7](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.mcp.json#L7) — `RAFA_MCP_KEY`
[7] [.claude/settings.json:5](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/settings.json#L5) — `@rafinery/cli`
[8] [.claude/settings.json:4](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/settings.json#L4) — `Read(.rafa`
[9] [.claude/settings.json:10](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/settings.json#L10) — `SessionStart`
[10] [.agents/skills/tdd/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.agents/skills/tdd/SKILL.md#L2) — `tdd`
[11] [.agents/skills/vercel-composition-patterns/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.agents/skills/vercel-composition-patterns/SKILL.md#L2) — `vercel-composition-patterns`
[12] [.claude/rafa/contract.md:642](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/rafa/contract.md#L642) — `The query plane`
[13] [.claude/rafa/contract.md:617](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/.claude/rafa/contract.md#L617) — `query_schema`
[14] [rafa.json:11](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/rafa.json#L11) — `skills`
[15] [rafa.json:8](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/rafa.json#L8) — `0.18.2`

<!-- okf:citations:end -->
