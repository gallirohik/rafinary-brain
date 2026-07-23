---
schemaVersion: 1
id: model-selection-flow
type: flow
domain: agent-bridge
title: "How the model dropdown actually selects an LLM (it's state, not the agent name)"
summary: "The ModelSelector writes ?coAgentsModel to the URL; that string becomes state.model and drives get_model on the backend — the agent NAME only branches google_genai vs not, and both names resolve to the same graph"
links: [agent-name-contract, langgraph-agent-convention, research-chat-flow]
cites:
  - src/components/ModelSelector.tsx:23 :: openai
  - src/lib/model-selector-provider.tsx:27 :: coAgentsModel
  - src/lib/model-selector-provider.tsx:42 :: research_agent
  - src/app/Main.tsx:12 :: model
  - agents/python/src/lib/model.py:18 :: state.get
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
`research_agent` and still get their model via `state.model`. So all four options do work; a
model failing is far more likely a missing provider API key (see
[env-and-integrations](/brain/rules/env-and-integrations.md)) than a wiring gap. Don't
"remove the dead options" — verify the keys first.

