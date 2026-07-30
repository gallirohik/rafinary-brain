---
schemaVersion: 1
id: stale-model-pins
priority: P1
category: ops
status: open
title: The anthropic dropdown option is already dead — claude-3-5-sonnet-20240620 was retired and now 404s
summary: "get_model pins claude-3-5-sonnet-20240620, which Anthropic retired on 2025-10-28 and which now returns 404 on every request, so selecting the anthropic model in the picker fails at call time today — not eventually; gemini-1.5-pro is in the same aging class and both pins are duplicated verbatim in the TypeScript port"
fix: "Replace the retired Anthropic pin with a current model id (claude-sonnet-5 is the documented replacement for this snapshot), re-check the Gemini pin against Google's current list, and update agents/typescript/src/model.ts in the same edit (~10 min)"
leverage: { impact: high, effort: low }
blast_radius: [agent-python, agent-typescript, external-integrations]
cites:
  - agents/python/src/lib/model.py:32 :: claude-3-5-sonnet-20240620
  - agents/python/src/lib/model.py:41 :: gemini-1.5-pro
  - agents/typescript/src/model.ts:26 :: claude-3-5-sonnet-20240620
  - agents/typescript/src/model.ts:32 :: gemini-1.5-pro
found: 2026-07-20
type: Improvement
description: "get_model pins claude-3-5-sonnet-20240620, which Anthropic retired on 2025-10-28 and which now returns 404 on every request, so selecting the anthropic model in the picker fails at call time today — not eventually; gemini-1.5-pro is in the same aging class and both pins are duplicated verbatim in the TypeScript port"
timestamp: 2026-07-20
tags: [ops, P1]
---
**Re-assessed 2026-07-30: escalated P3 → P1. The predicted failure has already happened.**

This row was opened on 2026-07-20 as a "will eventually deprecate" watch item. Checked against
Anthropic's current model catalog this pass, `claude-3-5-sonnet-20240620` (`model.py:32`) is
listed as **retired on 2025-10-28** — retired models return `404 not_found_error`, and the
documented drop-in replacement for this exact snapshot is `claude-sonnet-5`. So the `anthropic`
option in the picker ([model-selection-flow](/brain/playbooks/model-selection-flow.md)) has
been broken for roughly nine months, in both backends. The original body's framing — "low
urgency; tracked so it does not silently rot into a broken option" — is what has to change:
it already rotted.

`gemini-1.5-pro` (`model.py:41`) is the same class of pin and of the same vintage. Treat it as
suspect and re-check it against Google's current model list before shipping; this row does not
assert a retirement date for it, because that would be a claim without a source.

## Why this is P1 and not a nit

The failure mode is precisely the one this repo has no defence against. There are no tests, no
CI and no mypy ([coverage](/brain/coverage.md)), the model id is a plain string with no
validation, and nothing calls the provider at startup — so the 404 surfaces only when a user
picks that option mid-conversation. Worse, the model-selection playbook teaches readers that a
failing dropdown option is *usually a key/config problem*, which sends debugging at the wrong
target: the wiring is correct, the key is fine, the id is dead. A one-line fix that recovers a
quarter of the product's advertised surface is about as high-leverage as this ledger gets.

## Fix both ports in one edit

The pins are duplicated verbatim in the TypeScript agent — `model.ts:26` and `model.ts:32`
carry the same two strings ([agent-typescript-parity](/brain/rules/agent-typescript-parity.md)).
The Python agent is the one `npm run dev` actually starts, so fix it first, but change both in
the same commit or the parity note gains another drift entry.

Two things to get right while editing:

- **Use exact ids from the provider's current catalog; do not construct one.** In particular,
  do not append a date suffix to a current Anthropic alias — `claude-sonnet-5` is complete as
  written, and a hand-built `claude-sonnet-5-<date>` 404s exactly like the string being
  replaced.
- **A retired-model 404 is indistinguishable from a typo'd-model 404.** Since neither backend
  validates the id, consider whether the `raise ValueError("Invalid model specified")` path
  (`model.py:49`) should be joined by a startup check — out of scope for the 10-minute fix,
  but it is why this keeps being invisible.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/model.py:32](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/model.py#L32) — `claude-3-5-sonnet-20240620`
[2] [agents/python/src/lib/model.py:41](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/python/src/lib/model.py#L41) — `gemini-1.5-pro`
[3] [agents/typescript/src/model.ts:26](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/model.ts#L26) — `claude-3-5-sonnet-20240620`
[4] [agents/typescript/src/model.ts:32](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/agents/typescript/src/model.ts#L32) — `gemini-1.5-pro`

<!-- okf:citations:end -->
