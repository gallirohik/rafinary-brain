---
schemaVersion: 1
id: langgraph-agent-convention
type: convention
domain: agent-python
title: "LangGraph research agent — node structure, no-op tool schemas, and model routing"
summary: "The Python agent is a 6-node StateGraph (download→chat→search/delete/fact-check loop); tools are empty @tool schemas the model CALLS but the node handles; the concrete LLM is chosen at runtime by get_model from state.model. The TS agent still mirrors the older 5-node shape (fact_check_node not yet ported, verifiable-report plan decision)"
links: [agent-typescript-parity, agent-state-shape-contract, delete-resources-hitl-contract, research-chat-flow, env-and-integrations]
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
`fact_check_node`, which resolves the tool call itself (mirrors `search_node`'s pattern, not
the emit-config pattern). When adding a tool, follow
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
`download.py`'s `get_resource`) rather than re-fetching.
