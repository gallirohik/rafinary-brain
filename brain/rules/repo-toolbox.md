---
schemaVersion: 1
id: repo-toolbox
type: convention
domain: toolbox
title: The committed agent toolbox — rafa skills, the rafinery MCP server, and permissions
summary: The repo ships the rafa engineering SOP as .claude skills + 5 agent cards + the /rafa command, wired to the rafinery knowledge MCP over HTTP with a bearer key from env; permissions pre-allow the rafa CLI and .rafa reads
links: [env-and-integrations]
cites:
  - .claude/skills/rafa-scan/SKILL.md:2 :: rafa-scan
  - .claude/commands/rafa.md:2 :: version
  - .mcp.json:3 :: rafinery
  - .mcp.json:7 :: RAFA_MCP_KEY
  - .claude/settings.json:5 :: @rafinery/cli
  - .claude/settings.json:4 :: Read(.rafa
---
The only committed toolbox in this repo is **rafa** itself (the engineering SOP layer);
there is no app-specific tooling. This note is the index to it.

**Skills** (`.claude/skills/rafa-*/SKILL.md`, 10 of them): `rafa-build`, `rafa-distill`,
`rafa-improve`, `rafa-insights`, `rafa-leverage`, `rafa-okf`, `rafa-plan`, `rafa-sage`,
`rafa-scan`, `rafa-validate`. Each is a procedure card with `name:` ==
its directory (`rafa-scan/SKILL.md:2`) and a one-line `description:`. Invoke via the Skill
tool by name.

**Command**: `.claude/commands/rafa.md` — the `/rafa <verb>` slash command, `version: 2.0.2`
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
hooks (SessionStart digest, PostToolUse dirty-sensor, UserPromptSubmit reflex) and a rafa
status line — all pointing at `.claude/rafa/hooks/*.mjs`. Provider/service keys are covered
in [env-and-integrations](/brain/rules/env-and-integrations.md).
