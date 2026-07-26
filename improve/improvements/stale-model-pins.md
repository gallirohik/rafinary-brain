---
schemaVersion: 1
id: stale-model-pins
priority: P3
category: ops
status: open
title: Provider model IDs are pinned to aging snapshots that will eventually deprecate
summary: get_model pins claude-3-5-sonnet-20240620 and gemini-1.5-pro — dated snapshots that providers eventually retire, at which point the corresponding dropdown option starts failing with a provider 404/deprecation error
fix: Refresh the anthropic/google model IDs to current snapshots (and keep the TS port in sync) (~10 min)
leverage: { impact: low, effort: low }
blast_radius: [agent-python, external-integrations]
cites:
  - agents/python/src/lib/model.py:32 :: claude-3-5-sonnet-20240620
  - agents/python/src/lib/model.py:41 :: gemini-1.5-pro
found: 2026-07-20
type: Improvement
description: "get_model pins claude-3-5-sonnet-20240620 and gemini-1.5-pro — dated snapshots that providers eventually retire, at which point the corresponding dropdown option starts failing with a provider 404/deprecation error"
timestamp: 2026-07-20
---
`get_model` ([langgraph-agent-convention](/brain/rules/langgraph-agent-convention.md)) pins
`claude-3-5-sonnet-20240620` (`model.py:32`) and `gemini-1.5-pro` (`model.py:41`). These are
older dated snapshots; providers deprecate and eventually remove such versions, at which point the
`anthropic` and `google_genai` dropdown options ([model-selection-flow]
(/brain/playbooks/model-selection-flow.md)) will fail at call time with a provider-side error
even though the wiring is correct — a failure the model-selection playbook explicitly warns is
usually a key/config issue, not wiring. Refresh to current model IDs and keep the TS port
(`agents/typescript/src/model.ts`) in lockstep ([agent-typescript-parity]
(/brain/rules/agent-typescript-parity.md)). Verify the exact current IDs against each provider's
docs before changing. Low urgency; tracked so it does not silently rot into a broken option.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/model.py:32](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/model.py#L32) — `claude-3-5-sonnet-20240620`
[2] [agents/python/src/lib/model.py:41](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/model.py#L41) — `gemini-1.5-pro`

<!-- okf:citations:end -->
