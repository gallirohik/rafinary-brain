---
id: parallel-tool-calls-openai-only
type: Improvement
schemaVersion: 1
priority: P2
category: correctness
status: open
title: parallel_tool_calls is disabled for ChatOpenAI only, but chat_node answers just tool_calls[0] for every provider
summary: "chat_node suppresses parallel tool calls by class-name check on ChatOpenAI alone, then handles the response by reading tool_calls[0] and emitting exactly one ToolMessage; on anthropic/google_genai/grok the model may return several tool calls in one message, leaving the rest without a matching tool result and the next turn rejected by the provider"
fix: "Disable parallel tool calls for the other providers too (ChatAnthropic accepts it via bind_tools, Gemini via tool_config), or emit a ToolMessage for every entry in tool_calls rather than only [0] (~15 min)"
leverage: { impact: medium, effort: low }
blast_radius: [agent-python, agent-bridge]
cites:
  - agents/python/src/lib/chat.py:79 :: ChatOpenAI
  - agents/python/src/lib/chat.py:80 :: parallel_tool_calls
  - agents/python/src/lib/chat.py:120 :: tool_calls[0]
  - agents/python/src/lib/chat.py:129 :: tool_calls[0]
  - agents/python/src/lib/chat.py:153 :: tool_calls[0]
  - agents/python/src/lib/model.py:30 :: ChatAnthropic
  - agents/python/src/lib/model.py:39 :: ChatGoogleGenerativeAI
found: 2026-07-30
description: "chat_node suppresses parallel tool calls by class-name check on ChatOpenAI alone, then handles the response by reading tool_calls[0] and emitting exactly one ToolMessage; on anthropic/google_genai/grok the model may return several tool calls in one message, leaving the rest without a matching tool result and the next turn rejected by the provider"
tags: [correctness, P2]
timestamp: 2026-07-30
---
`chat_node` builds its invoke kwargs with a class-name test:

```
if model.__class__.__name__ in ["ChatOpenAI"]:
    ainvoke_kwargs["parallel_tool_calls"] = False
```

(`chat.py:79-80`). The list has exactly one entry, but the picker offers four backends —
`ChatOpenAI`, `ChatAnthropic` (`model.py:30`), `ChatGoogleGenerativeAI` (`model.py:39`) and
`ChatXAI` ([model-selection-flow](/brain/playbooks/model-selection-flow.md)). For the other
three, nothing suppresses parallel tool calls.

The rest of the node is written as if exactly one can ever arrive. Every branch indexes
`ai_message.tool_calls[0]` — the `WriteReport` handler (`chat.py:120`), its `ToolMessage`
(`chat.py:129`), and the routing chain that picks `search_node` / `delete_node` /
`fact_check_node` (`chat.py:153`). So when a model returns two tool calls in one assistant
message, the first is answered and the second is silently dropped: the conversation now holds
a `tool_use` with no matching `tool_result`, which Anthropic and Google both reject on the
**next** request, not this one.

That delayed rejection is what makes it worth a row. It is not a crash at the call site — it
is a provider-side 400 one turn later, on a message the user did not send, which reads as "the
chat broke for no reason" and only on some model choices. Nothing in the repo catches it:
there are no tests, no CI, and no mypy ([coverage](/brain/coverage.md) — the test surface is
grep-proven absent), and the happy path is the single-tool-call case the system prompt already
nudges the model toward.

## Two ways to close it, pick one

- **Suppress it everywhere** (smaller diff, matches current intent). Widen the check to the
  other classes; each provider has its own spelling of the flag, so this is a per-provider
  mapping rather than one shared kwarg — verify against the installed LangChain integration
  before wiring, don't assume `parallel_tool_calls` is accepted verbatim.
- **Handle the list** (more honest). Iterate `ai_message.tool_calls` and emit one
  `ToolMessage` per entry, keeping the routing decision on `[0]`. This also makes the node
  correct if the suppression flag is ever dropped or ignored by a provider.

Related but distinct: `fact_check_node` indexes `tool_calls[0]` with no *existence* guard at
all ([fact-check-node-unguarded-toolcall](/improve/improvements/fact-check-node-unguarded-toolcall.md)).
That row is about the empty case; this one is about the many case. A single sitting in
`chat.py`/`fact_check.py` can settle both.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/lib/chat.py:79](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L79) — `ChatOpenAI`
[2] [agents/python/src/lib/chat.py:80](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L80) — `parallel_tool_calls`
[3] [agents/python/src/lib/chat.py:120](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L120) — `tool_calls[0]`
[4] [agents/python/src/lib/chat.py:129](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L129) — `tool_calls[0]`
[5] [agents/python/src/lib/chat.py:153](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/chat.py#L153) — `tool_calls[0]`
[6] [agents/python/src/lib/model.py:30](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L30) — `ChatAnthropic`
[7] [agents/python/src/lib/model.py:39](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/model.py#L39) — `ChatGoogleGenerativeAI`

<!-- okf:citations:end -->
