---
schemaVersion: 1
id: model-selection-flow
type: flow
domain: agent-bridge
title: "How the model dropdown actually selects an LLM (it's state, not the agent name)"
summary: "The ModelSelector writes ?coAgentsModel to the URL; that string becomes state.model and drives get_model on the backend — the agent NAME only branches google_genai vs not, and both names resolve to the same graph. The wiring is correct for all four options, but the anthropic MODEL ID is dead (claude-3-5-sonnet-20240620, retired 2025-10-28): check the id before the key"
links: [agent-name-contract, langgraph-agent-convention, research-chat-flow]
cites:
  - src/components/ModelSelector.tsx:23 :: openai
  - src/lib/model-selector-provider.tsx:27 :: coAgentsModel
  - src/lib/model-selector-provider.tsx:42 :: research_agent
  - src/app/Main.tsx:12 :: model
  - agents/python/src/lib/model.py:18 :: state.get
  - agents/python/src/lib/model.py:32 :: claude-3-5-sonnet-20240620
  - agents/typescript/src/model.ts:26 :: claude-3-5-sonnet-20240620
  - agents/python/src/lib/model.py:41 :: gemini-1.5-pro
  - agents/python/src/lib/model.py:49 :: raise ValueError
description: "The ModelSelector writes ?coAgentsModel to the URL; that string becomes state.model and drives get_model on the backend — the agent NAME only branches google_genai vs not, and both names resolve to the same graph. The wiring is correct for all four options, but the anthropic MODEL ID is dead (claude-3-5-sonnet-20240620, retired 2025-10-28): check the id before the key"
tags: [agent-bridge]
timestamp: 2026-07-26T23:55:19Z
---
A subtle flow worth getting right before you "fix a broken model option."

1. The dropdown offers four values — `openai`, `anthropic`, `google_genai`, `grok`
   (`ModelSelector.tsx:23-26`). Selecting one calls `setModel`, which sets the
   `?coAgentsModel=` URL param and reloads (`model-selector-provider.tsx:31-35`).

2. On load, the provider reads `coAgentsModel` (default `openai`) into `model`
   (`model-selector-provider.tsx:27`), and derives the **agent name**: `research_agent` for
   everything, switching to `research_agent_google_genai` **only** when
   `model === "google_genai"` (`:42-45`).

3. `model` is seeded into the coagent's `initialState.model` via the shared
   `createInitialAgentState(model)` factory (`Main.tsx:12`,
   `ResearchCanvas.tsx` — both call sites now use the same factory, see
   [the state shape contract](/brain/rules/agent-state-shape-contract.md)),
   so it travels to the backend as `state.model`.

4. The backend's `get_model` reads `state.get("model", "openai")` (overridable by the `MODEL`
   env var) and constructs the matching LangChain chat model (`model.py:18-49`). **This** is
   where the choice takes effect — not the agent name.

The non-obvious part: both `research_agent` and `research_agent_google_genai` map to the
**same** compiled graph (see [langgraph.json](/brain/rules/agent-name-contract.md)), so the
distinct google_genai agent name is essentially cosmetic — `anthropic` and `grok` run through
`research_agent` and still get their model via `state.model`. Nothing in the picker is dead
*wiring*, so "remove the dead options" is the wrong first move.

## Wired is not working — check the model ID before the key

The wiring above is correct for all four options. Whether the option **works** is a separate
question, decided one line lower, by the model id string `get_model` hands the provider:

- `anthropic` is **broken today**, and has been for roughly nine months. The branch pins
  `claude-3-5-sonnet-20240620` (`model.py:32`, duplicated verbatim in the TS port at
  `model.ts:26`), an id Anthropic retired on 2025-10-28. Retired ids return
  `404 not_found_error`, so this option fails at call time with a perfectly valid key and
  perfectly correct wiring — see the open **P1**
  [stale-model-pins](/improve/improvements/stale-model-pins.md), which carries the fix.
- `google_genai` pins `gemini-1.5-pro` (`model.py:41`) — same vintage, same class of pin.
  This note asserts no retirement date for it; treat it as suspect and re-check it against
  Google's current list before relying on it.

**So the debug order is: id first, key second.** A dead-id 404 and a missing-key 401 both
surface identically as "that dropdown option is broken", and nothing in this repo tells them
apart for you: the id is a plain string, the only validation is the final
`raise ValueError` for an *unknown provider name* (`model.py:49`) — never for an unknown
model id — nothing calls the provider at startup, and there are no tests, no mypy and no CI.
Confirm the pinned id appears in the provider's **current** catalogue first; only then go to
[env-and-integrations](/brain/rules/env-and-integrations.md) for the key.

(Earlier revisions of this note closed with "verify the keys first". That was wrong in the
load-bearing direction and is why the ledger row above names this playbook by id: it sent
debugging at the key while the id was the dead thing.)

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/components/ModelSelector.tsx:23](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/components/ModelSelector.tsx#L23) — `openai`
[2] [src/lib/model-selector-provider.tsx:27](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/model-selector-provider.tsx#L27) — `coAgentsModel`
[3] [src/lib/model-selector-provider.tsx:42](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/lib/model-selector-provider.tsx#L42) — `research_agent`
[4] [src/app/Main.tsx:12](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/Main.tsx#L12) — `model`
[5] [agents/python/src/lib/model.py:18](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L18) — `state.get`
[6] [agents/python/src/lib/model.py:32](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L32) — `claude-3-5-sonnet-20240620`
[7] [agents/typescript/src/model.ts:26](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/typescript/src/model.ts#L26) — `claude-3-5-sonnet-20240620`
[8] [agents/python/src/lib/model.py:41](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L41) — `gemini-1.5-pro`
[9] [agents/python/src/lib/model.py:49](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L49) — `raise ValueError`

<!-- okf:citations:end -->
