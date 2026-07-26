---
schemaVersion: 1
id: repo-toolbox
type: convention
domain: toolbox
title: The committed agent toolbox — rafa skills, harness-neutral .agents skills, the rafinery MCP server, and permissions
summary: The repo ships the rafa engineering SOP as 13 .claude skills + 5 agent cards + the /rafa command, PLUS 7 harness-neutral skills under .agents/skills (Claude Code · Codex · Cursor all read these), wired to the rafinery knowledge MCP over HTTP with a bearer key from env
links: [env-and-integrations]
cites:
  - .claude/skills/rafa-scan/SKILL.md:2 :: rafa-scan
  - .claude/commands/rafa.md:2 :: version
  - .mcp.json:3 :: rafinery
  - .mcp.json:7 :: RAFA_MCP_KEY
  - .claude/settings.json:5 :: @rafinery/cli
  - .claude/settings.json:4 :: Read(.rafa
  - .claude/settings.json:10 :: SessionStart
  - .agents/skills/tdd/SKILL.md:2 :: tdd
  - .agents/skills/vercel-composition-patterns/SKILL.md:2 :: vercel-composition-patterns
  - rafa.json:11 :: skills
description: "The repo ships the rafa engineering SOP as 13 .claude skills + 5 agent cards + the /rafa command, PLUS 7 harness-neutral skills under .agents/skills (Claude Code · Codex · Cursor all read these), wired to the rafinery knowledge MCP over HTTP with a bearer key from env"
tags: [toolbox]
timestamp: 2026-07-26T22:44:42.840Z
---
There is no app-specific tooling in this repo — the committed toolbox is **rafa** (the
engineering SOP layer) plus a set of harness-neutral skill dependencies it installed. This
note is the index to both. Check here before hand-rolling a procedure.

**rafa skills** (`.claude/skills/rafa-*/SKILL.md`, **13**): `rafa-build`, `rafa-commit`,
`rafa-distill`, `rafa-improve`, `rafa-insights`, `rafa-leverage`, `rafa-okf`, `rafa-plan`,
`rafa-review`, `rafa-sage`, `rafa-scan`, `rafa-security`, `rafa-validate`. Each is a
procedure card with `name:` == its directory (`rafa-scan/SKILL.md:2`) and a one-line
`description:`. Invoke via the Skill tool by name.

**Harness-neutral skills** (`.agents/skills/*/SKILL.md`, **7**): `tdd`, `frontend-design`,
`vercel-composition-patterns`, `requesting-code-review`, `improve-codebase-architecture`,
`grill-me`, `grilling`. Same `name:`/`description:` frontmatter shape
(`.agents/skills/tdd/SKILL.md:2`,
`.agents/skills/vercel-composition-patterns/SKILL.md:2`). These live outside `.claude/`
**on purpose** — `.agents/` is the one convention Claude Code, Codex and Cursor all read,
so the same skill works whichever harness a teammate uses. They are consent-installed by
`rafa update` and their versions are recorded in `rafa.json`'s
`rafa.skills.installed` map (`rafa.json:11`), with a parallel `declined: []` list. Adding a
skill means installing it there AND recording the version in `rafa.json` — an
`.agents/skills/` directory with no `rafa.json` entry is un-tracked drift.

**Command**: `.claude/commands/rafa.md` — the `/rafa <verb>` slash command, `version: 2.2.0`
(`:2`).

**Agent cards** (`.claude/agents/*.md`, 5): `atlas` (scan/build/plan), `bloom` (improve),
`prism` (validate), `sage` (learnings), `compass` (user insights). These are contract-gated
(§10) but are workforce config, not app code.

**MCP server** (`.mcp.json`): one server, `rafinery` (`:3`), `type: http`, url
`https://dev.rafinery.ai/api/mcp`, authenticated `Authorization: Bearer ${RAFA_MCP_KEY}`
(`:7`) — the env var is expanded at load; the raw key lives in gitignored
`.claude/settings.local.json` / `~/.config/rafinery/credentials.json`, never committed. This
is the read-only knowledge MCP over the ingested brain (search/get/ask).

**Permissions** (`.claude/settings.json`): pre-allows `Read(.rafa/**)` (`:4`),
`Bash(npx @rafinery/cli:*)` (`:5`), `Bash(rafa:*)` (`:6`). The file also registers three
hooks — `SessionStart` digest (`:10`, matcher `startup|resume|clear`), `PostToolUse`
dirty-sensor (`:21`, matcher `Edit|Write|MultiEdit|NotebookEdit`), `UserPromptSubmit` reflex
(`:32`) — and a rafa status line (`:44`), all pointing at `.claude/rafa/hooks/*.mjs`.
Alongside them, `.claude/rafa/hooks/` also ships the git-side hooks (`post-commit`,
`pre-push`, `post-checkout`, `post-rewrite`) that mirror and checkpoint the brain; those
are installed into `.git/hooks`, not declared in `settings.json`.
Provider/service keys are covered in
[env-and-integrations](/brain/rules/env-and-integrations.md).

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [.claude/skills/rafa-scan/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.claude/skills/rafa-scan/SKILL.md#L2) — `rafa-scan`
[2] [.claude/commands/rafa.md:2](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.claude/commands/rafa.md#L2) — `version`
[3] [.mcp.json:3](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.mcp.json#L3) — `rafinery`
[4] [.mcp.json:7](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.mcp.json#L7) — `RAFA_MCP_KEY`
[5] [.claude/settings.json:5](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.claude/settings.json#L5) — `@rafinery/cli`
[6] [.claude/settings.json:4](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.claude/settings.json#L4) — `Read(.rafa`
[7] [.claude/settings.json:10](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.claude/settings.json#L10) — `SessionStart`
[8] [.agents/skills/tdd/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.agents/skills/tdd/SKILL.md#L2) — `tdd`
[9] [.agents/skills/vercel-composition-patterns/SKILL.md:2](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/.agents/skills/vercel-composition-patterns/SKILL.md#L2) — `vercel-composition-patterns`
[10] [rafa.json:11](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/rafa.json#L11) — `skills`

<!-- okf:citations:end -->
