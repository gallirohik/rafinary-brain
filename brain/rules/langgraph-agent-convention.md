---
schemaVersion: 1
id: langgraph-agent-convention
type: convention
domain: agent-python
title: LangGraph research agent — node structure, no-op tool schemas, and model routing
summary: Both agents are the same 5-node StateGraph (download→chat→search/delete loop); tools are empty @tool schemas the model CALLS but the node handles; the concrete LLM is chosen at runtime by get_model from state.model
links: [agent-typescript-parity, agent-state-shape-contract, delete-resources-hitl-contract, research-chat-flow, env-and-integrations]
cites:
  - agents/python/src/agent.py:17 :: StateGraph
  - agents/python/src/agent.py:25 :: set_entry_point
  - agents/python/src/agent.py:33 :: interrupt_after
  - agents/python/src/agent.py:37 :: LANGGRAPH_FASTAPI
  - agents/python/src/lib/chat.py:16 :: @tool
  - agents/python/src/lib/chat.py:77 :: bind_tools
  - agents/python/src/lib/model.py:23 :: openai
  - agents/python/src/lib/model.py:49 :: raise ValueError
  - agents/python/src/lib/search.py:35 :: TavilyClient
  - agents/python/src/lib/download.py:30 :: _is_safe_url
---
The Python agent (`agents/python/`, the primary/actively-run backend — `npm run dev` starts
it) is a LangGraph `StateGraph` over [AgentState](/brain/rules/agent-state-shape-contract.md).
The TS agent mirrors it; see [agent parity](/brain/rules/agent-typescript-parity.md).

**Graph shape** (`agent.py:17-33`): five nodes — `download`, `chat_node`, `search_node`,
`delete_node`, `perform_delete_node`. Entry is `download` (`:25`). Flow:
`download → chat_node`; `chat_node` branches (to `search_node`, `delete_node`, back to
itself, or END); `search_node → download` (loop to fetch new resources);
`delete_node → perform_delete_node → chat_node`. Compiled with
`interrupt_after=["delete_node"]` (`:33`) — the human-in-the-loop pause, see
[the DeleteResources contract](/brain/rules/delete-resources-hitl-contract.md).

**The no-op tool idiom** (`chat.py:16-38`): tools (`Search`, `WriteReport`,
`WriteResearchQuestion`, `DeleteResources`, and `ExtractResources` in search) are declared
as empty `@tool` functions — they have **no body**. They exist only as *schemas* the model
emits tool-calls against; the node code inspects `ai_message.tool_calls[0]["name"]` and acts.
`WriteReport`/`WriteResearchQuestion` args are streamed into state via
`copilotkit_customize_config(emit_intermediate_state=...)` (`chat.py:43`). When adding a
tool, follow [the add-tool how-to](/brain/playbooks/add-agent-tool-howto.md).

**Model routing** (`model.py`): `get_model(state)` reads `state.model` (overridable by the
`MODEL` env var) and returns a LangChain chat model — `openai`→gpt-4o-mini,
`anthropic`→claude-3-5-sonnet, `google_genai`→gemini-1.5-pro, `grok`→grok-4 (`:23-47`),
else `raise ValueError` (`:49`). Provider packages are imported lazily inside each branch.
Note `ChatOpenAI` gets `parallel_tool_calls=False` specifically (`chat.py`/`search.py`).

**External calls**: web search via `TavilyClient` (`search.py:35`); resource fetch in
`download.py` runs HTML→markdown (`html2text`) behind an **SSRF guard** `_is_safe_url`
(`download.py:30`) that rejects private/loopback/link-local IPs. Results are memoized in the
in-process `_RESOURCE_CACHE`.
