---
schemaVersion: 1
id: langgraph-agent-convention
type: convention
domain: agent-python
title: "LangGraph research agent — node structure, no-op tool schemas, and model routing"
summary: "The Python agent is a 6-node StateGraph (download→chat→search/delete/fact-check loop); tools are empty @tool schemas the model CALLS but the node handles; the concrete LLM is chosen at runtime by get_model from state.model. Two scars live here: the guard-tool_calls[0] convention that fact_check_node itself breaks at 4 sites, and the \"ERROR\" sentinel in _RESOURCE_CACHE that makes one failed download permanent for the process. The TS agent still mirrors the older 5-node shape (fact_check_node not yet ported, verifiable-report plan decision)"
links: [agent-typescript-parity, agent-state-shape-contract, delete-resources-hitl-contract, research-chat-flow, env-and-integrations, security-posture]
cites:
  - agents/python/src/agent.py:18 :: StateGraph
  - agents/python/src/agent.py:27 :: set_entry_point
  - agents/python/src/agent.py:36 :: interrupt_after
  - agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
  - agents/python/src/agent.py:13 :: fact_check_node
  - agents/python/src/agent.py:24 :: fact_check_node
  - agents/python/src/agent.py:31 :: fact_check_node
  - agents/python/src/lib/chat.py:37 :: FactCheckReport
  - agents/python/src/lib/chat.py:88 :: FactCheckReport
  - agents/python/src/lib/chat.py:160 :: FactCheckReport
  - agents/python/src/lib/fact_check.py:44 :: ExtractClaimChecks
  - agents/python/src/lib/model.py:23 :: openai
  - agents/python/src/lib/model.py:49 :: raise ValueError
  - agents/python/src/lib/search.py:35 :: TavilyClient
  - agents/python/src/lib/download.py:30 :: _is_safe_url
  - agents/python/src/lib/search.py:67 :: if not ai_message.tool_calls
  - agents/python/src/lib/search.py:139 :: if not ai_message_response.tool_calls
  - agents/python/src/lib/delete.py:26 :: if ai_message.tool_calls
  - agents/python/src/lib/fact_check.py:65 :: ai_message.tool_calls[0]["id"]
  - agents/python/src/lib/fact_check.py:114 :: ai_message.tool_calls[0]["id"]
  - agents/python/src/lib/fact_check.py:126 :: if not ai_message_response.tool_calls
  - agents/python/src/lib/fact_check.py:129 :: ai_message.tool_calls[0]["id"]
  - agents/python/src/lib/fact_check.py:141 :: ai_message.tool_calls[0]["id"]
  - agents/python/src/lib/download.py:24 :: _RESOURCE_CACHE.get(url, "")
  - agents/python/src/lib/download.py:66 :: _RESOURCE_CACHE[url] = "ERROR"
  - agents/python/src/lib/download.py:81 :: _RESOURCE_CACHE[url] = "ERROR"
  - agents/python/src/lib/download.py:97 :: if not get_resource(resource["url"])
  - agents/python/src/lib/chat.py:72 :: if content == "ERROR"
  - agents/python/src/lib/fact_check.py:81 :: if content in ("", "ERROR")
description: "The Python agent is a 6-node StateGraph (download→chat→search/delete/fact-check loop); tools are empty @tool schemas the model CALLS but the node handles; the concrete LLM is chosen at runtime by get_model from state.model. Two scars live here: the guard-tool_calls[0] convention that fact_check_node itself breaks at 4 sites, and the \'ERROR\' sentinel in _RESOURCE_CACHE that makes one failed download permanent for the process. The TS agent still mirrors the older 5-node shape (fact_check_node not yet ported, verifiable-report plan decision)"
tags: [agent-python]
timestamp: 2026-07-26T22:44:42.840Z
---
The Python agent (`agents/python/`, the primary/actively-run backend — `npm run dev` starts
it) is a LangGraph `StateGraph` over [AgentState](/brain/rules/agent-state-shape-contract.md).
The TS agent mirrors an OLDER version of this graph; see
[agent parity](/brain/rules/agent-typescript-parity.md) — it does not yet have
`fact_check_node` (a logged plan decision, "Python-only for now").

**Graph shape** (`agent.py:18-31`): six nodes — `download`, `chat_node`, `search_node`,
`delete_node`, `perform_delete_node`, `fact_check_node`. Entry is `download` (`:27`). Flow:
`download → chat_node`; `chat_node` branches (to `search_node`, `delete_node`,
`fact_check_node`, back to itself, or END); `search_node → download` (loop to fetch new
resources); `delete_node → perform_delete_node → chat_node`; `fact_check_node → chat_node`
(loops back once fact-checking is done, same shape as `perform_delete_node`). Compiled with
`interrupt_after=["delete_node"]` (`:36`) — the human-in-the-loop pause, see
[the DeleteResources contract](/brain/rules/delete-resources-hitl-contract.md). No interrupt
on `fact_check_node` — it needs no human confirmation, unlike delete.

**The no-op tool idiom** (`chat.py:16-38`): tools (`Search`, `WriteReport`,
`WriteResearchQuestion`, `DeleteResources`, `FactCheckReport`, and internal extraction tools
`ExtractResources`/`ExtractClaimChecks` in `search.py`/`fact_check.py`) are declared as empty
`@tool` functions — they have **no body**. They exist only as *schemas* the model emits
tool-calls against; the node code inspects `ai_message.tool_calls[0]["name"]` and acts.
`WriteReport`/`WriteResearchQuestion` args are streamed into state via
`copilotkit_customize_config(emit_intermediate_state=...)` (`chat.py:48`). `FactCheckReport`
takes no args (the whole report is already in state) and routes straight to
`fact_check_node`, which resolves the tool call itself rather than using the emit-config
pattern. **On the *resolution* axis only** that is `search_node`'s shape — on the *guarding*
axis `fact_check_node` is a **non-exemplar**; do not use it as the template (see the
defensive convention at the bottom of this note). When adding a tool, follow
[the add-tool how-to](/brain/playbooks/add-agent-tool-howto.md).

**Model routing** (`model.py`): `get_model(state)` reads `state.model` (overridable by the
`MODEL` env var) and returns a LangChain chat model — `openai`→gpt-4o-mini,
`anthropic`→claude-3-5-sonnet, `google_genai`→gemini-1.5-pro, `grok`→grok-4 (`:23-47`),
else `raise ValueError` (`:49`). Provider packages are imported lazily inside each branch.
Note `ChatOpenAI` gets `parallel_tool_calls=False` specifically (`chat.py`/`search.py`/
`fact_check.py`).

**External calls**: web search via `TavilyClient` (`search.py:35`); resource fetch in
`download.py` runs HTML→markdown (`html2text`) behind an **SSRF guard** `_is_safe_url`
(`download.py:30`) that rejects private/loopback/link-local IPs. Results are memoized in the
in-process `_RESOURCE_CACHE`; `fact_check_node` reads from this same cache (via
`download.py`'s `get_resource`) rather than re-fetching. The agent server itself is
unauthenticated — see [the security posture](/brain/playbooks/security-posture.md).

**Gotcha — `"ERROR"` is a sentinel VALUE in `_RESOURCE_CACHE`, and it caches failure
permanently.** `_download_resource` writes the literal string `"ERROR"` into the cache on
**any** failure — SSRF rejection (`download.py:66`) or any exception at all, including a
transient network blip (`download.py:81`, a bare `except Exception`). Three consumers across
three files then branch on that value, and the branch that matters is a bug:

- `download_node`'s re-download test is `if not get_resource(resource["url"])`
  (`download.py:97`). `get_resource` returns `_RESOURCE_CACHE.get(url, "")`
  (`download.py:24`) — a **non-empty** string, so `"ERROR"` is **truthy** and the URL is
  never added to `resources_to_download` again. **A URL that fails once is never retried for
  the life of the process.** Only `""` (never cached) triggers a fetch.
- `chat_node` skips it: `if content == "ERROR": continue` (`chat.py:72`).
- `fact_check_node` skips it: `if content in ("", "ERROR"): continue` (`fact_check.py:81`).

Net effect a dev will actually hit: one transient failure poisons that resource for the
whole process — the card stays on the canvas, the progress log says **done**, the model
never sees the content, and no error surfaces anywhere in state or the UI. If someone
reports "my resource has no content" or "fact-check says nothing supports this claim,"
this is the first thing to check, and the check is a restart (the cache is module-level and
dies with the process). Two consequences for new code: a fourth consumer that forgets the
sentinel will feed the model the literal text `ERROR` as page content, and any fix must
change `download.py:97`'s truthiness test, not just the sentinel, or retries stay dead.

**Defensive convention: never index `tool_calls[0]` unguarded.** Forced `tool_choice` is not
honored identically by all four providers, so `search_node` checks `if not
...tool_calls:` before both the incoming-Search read (`search.py:67`, guarding `:74`) and the
forced `ExtractResources` response (`search.py:139`, guarding `:152`), logging into
`state["logs"]` and returning early instead of raising `IndexError`. The second guard **also
appends a `ToolMessage` resolving the original Search `tool_call_id`** (`search.py:145`) — an
unresolved tool call makes the NEXT `chat_node` turn 400, so an early return that skips it
just moves the failure one hop. Copy **`search_node`'s** shape in any new node.

**But the convention is NOT universally applied — 3 of the 4 consumer nodes guard, and the
newest one does not.** Check the code, not the convention, before copying a node:

| node | incoming `tool_calls[0]` read | guarded? |
| --- | --- | --- |
| `search_node` | `search.py:74` (queries) | ✓ `search.py:67` |
| `search_node` (forced response) | `search.py:152` (resources) | ✓ `search.py:139`, and it appends the resolving `ToolMessage` at `:145` |
| `perform_delete_node` | `delete.py:27` (urls) | ✓ `delete.py:26`, with an `additional_kwargs["function_call"]` fallback |
| `fact_check_node` | `fact_check.py:65`, `:114`, `:129`, `:141` | ✗ **all four unguarded** |

**`fact_check_node` is the non-exemplar.** Every one of its four
`ai_message.tool_calls[0]["id"]` reads (`fact_check.py:65`, `:114`, `:129`, `:141`) indexes
the *incoming* message with no `if not ...tool_calls:` check, so a provider that routes to
the node without an accompanying tool call raises `IndexError` inside the node. Its single
guard, `fact_check.py:126`, covers only `ai_message_response` — the *forced*
`ExtractClaimChecks` reply — and `:129` is the worst case: the unguarded incoming index sits
**inside** that guard's early-return path, i.e. the branch whose whole job is to resolve the
tool call. It is the newest node and therefore the most tempting template; copying it
reproduces exactly the `IndexError` class `search_node` already fixed. Fixing
`fact_check_node` (hoist `ai_message.tool_calls[0]["id"]` into one guarded local at the top
of the node) is a code change, not a brain change — until it lands, this note is the warning.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [agents/python/src/agent.py:18](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L18) — `StateGraph`
[2] [agents/python/src/agent.py:27](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L27) — `set_entry_point`
[3] [agents/python/src/agent.py:36](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L36) — `interrupt_after`
[4] [agents/python/src/agent.py:40](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L40) — `LANGGRAPH_FASTAPI`
[5] [agents/python/src/agent.py:13](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L13) — `fact_check_node`
[6] [agents/python/src/agent.py:24](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L24) — `fact_check_node`
[7] [agents/python/src/agent.py:31](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/agent.py#L31) — `fact_check_node`
[8] [agents/python/src/lib/chat.py:37](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/chat.py#L37) — `FactCheckReport`
[9] [agents/python/src/lib/chat.py:88](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/chat.py#L88) — `FactCheckReport`
[10] [agents/python/src/lib/chat.py:160](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/chat.py#L160) — `FactCheckReport`
[11] [agents/python/src/lib/fact_check.py:44](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L44) — `ExtractClaimChecks`
[12] [agents/python/src/lib/model.py:23](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/model.py#L23) — `openai`
[13] [agents/python/src/lib/model.py:49](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/model.py#L49) — `raise ValueError`
[14] [agents/python/src/lib/search.py:35](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L35) — `TavilyClient`
[15] [agents/python/src/lib/download.py:30](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/download.py#L30) — `_is_safe_url`
[16] [agents/python/src/lib/search.py:67](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L67) — `if not ai_message.tool_calls`
[17] [agents/python/src/lib/search.py:139](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/search.py#L139) — `if not ai_message_response.tool_calls`
[18] [agents/python/src/lib/delete.py:26](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/delete.py#L26) — `if ai_message.tool_calls`
[19] [agents/python/src/lib/fact_check.py:65](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L65) — `ai_message.tool_calls[0]["id"]`
[20] [agents/python/src/lib/fact_check.py:114](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L114) — `ai_message.tool_calls[0]["id"]`
[21] [agents/python/src/lib/fact_check.py:126](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L126) — `if not ai_message_response.tool_calls`
[22] [agents/python/src/lib/fact_check.py:129](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L129) — `ai_message.tool_calls[0]["id"]`
[23] [agents/python/src/lib/fact_check.py:141](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L141) — `ai_message.tool_calls[0]["id"]`
[24] [agents/python/src/lib/download.py:24](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/download.py#L24) — `_RESOURCE_CACHE.get(url, "")`
[25] [agents/python/src/lib/download.py:66](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/download.py#L66) — `_RESOURCE_CACHE[url] = "ERROR"`
[26] [agents/python/src/lib/download.py:81](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/download.py#L81) — `_RESOURCE_CACHE[url] = "ERROR"`
[27] [agents/python/src/lib/download.py:97](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/download.py#L97) — `if not get_resource(resource["url"])`
[28] [agents/python/src/lib/chat.py:72](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/chat.py#L72) — `if content == "ERROR"`
[29] [agents/python/src/lib/fact_check.py:81](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/agents/python/src/lib/fact_check.py#L81) — `if content in ("", "ERROR")`

<!-- okf:citations:end -->
