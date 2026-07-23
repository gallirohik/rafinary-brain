---
type: "Citation Check"
title: "Citation check report"
description: "checker v2 — all gates pass · 20 warn(s)"
timestamp: 2026-07-23T21:55:44.969Z
---
# Citation check (generated — do not hand-edit) · checker v2

## Resolution (B1): 43/43 ✓
✓ src/lib/types.ts:19 :: AgentState
✓ src/lib/types.ts:21 :: research_question
✓ src/lib/types.ts:22 :: report
✓ src/lib/types.ts:23 :: resources
✓ src/lib/types.ts:24 :: logs
✓ src/lib/types.ts:25 :: citations
✓ agents/python/src/lib/state.py:41 :: AgentState
✓ agents/python/src/lib/state.py:48 :: research_question
✓ agents/python/src/lib/state.py:51 :: logs
✓ agents/python/src/lib/state.py:52 :: citations
✓ agents/typescript/src/state.ts:19 :: AgentStateAnnotation
✓ agents/typescript/src/state.ts:21 :: research_question
✓ agents/typescript/src/state.ts:24 :: logs
✓ agents/python/src/agent.py:18 :: StateGraph
✓ agents/python/src/agent.py:27 :: set_entry_point
✓ agents/python/src/agent.py:36 :: interrupt_after
✓ agents/python/src/agent.py:40 :: LANGGRAPH_FASTAPI
✓ agents/python/src/agent.py:13 :: fact_check_node
✓ agents/python/src/agent.py:24 :: fact_check_node
✓ agents/python/src/agent.py:31 :: fact_check_node
✓ agents/python/src/lib/chat.py:37 :: FactCheckReport
✓ agents/python/src/lib/chat.py:88 :: FactCheckReport
✓ agents/python/src/lib/chat.py:160 :: FactCheckReport
✓ agents/python/src/lib/fact_check.py:44 :: ExtractClaimChecks
✓ agents/python/src/lib/model.py:23 :: openai
✓ agents/python/src/lib/model.py:49 :: raise ValueError
✓ agents/python/src/lib/search.py:35 :: TavilyClient
✓ agents/python/src/lib/download.py:30 :: _is_safe_url
✓ src/components/ModelSelector.tsx:23 :: openai
✓ src/lib/model-selector-provider.tsx:27 :: coAgentsModel
✓ src/lib/model-selector-provider.tsx:42 :: research_agent
✓ src/app/Main.tsx:12 :: model
✓ agents/python/src/lib/model.py:18 :: state.get
✓ src/app/Main.tsx:45 :: CopilotChat
✓ src/app/api/copilotkit/route.ts:105 :: handleRequest
✓ agents/python/src/lib/chat.py:82 :: bind_tools
✓ agents/python/src/lib/chat.py:48 :: copilotkit_customize_config
✓ agents/python/src/lib/chat.py:160 :: FactCheckReport
✓ agents/python/src/lib/fact_check.py:137 :: citations
✓ agents/python/src/lib/search.py:81 :: copilotkit_emit_state
✓ src/components/ResearchCanvas.tsx:27 :: useCoAgentStateRender
✓ src/components/ResearchCanvas.tsx:193 :: report
✓ src/components/ResearchCanvas.tsx:202 :: citations

## Completeness (B2): 0/0 ✓  (0 anchors)

## Policy (contract → anchor declared): 1/1 ✓
✓ rules/agent-state-shape-contract.md

## Absence (B3, declared `absent:` re-grepped): 0/0 ✓

## Inventory (coverage declared vs `git ls-files`): 0/0 ✓

## Warns (heuristic, non-failing — existence-shaped title/summary with no `absent:` declared): 0

## Links (non-failing, OKF §5.3 — dangling cross-links, bundle-wide resolution): 20
⚠ rules/agent-state-shape-contract.md — frontmatter → agent-name-contract does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/agent-state-shape-contract.md — frontmatter → copilotkit-runtime-route-convention does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/agent-state-shape-contract.md — mdlink → /brain/rules/agent-typescript-parity.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/agent-state-shape-contract.md — mdlink → /brain/rules/design-system-convention.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — frontmatter → agent-typescript-parity does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — frontmatter → delete-resources-hitl-contract does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — frontmatter → env-and-integrations does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — mdlink → /brain/rules/agent-typescript-parity.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — mdlink → /brain/rules/delete-resources-hitl-contract.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ rules/langgraph-agent-convention.md — mdlink → /brain/playbooks/add-agent-tool-howto.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/model-selection-flow.md — frontmatter → agent-name-contract does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/model-selection-flow.md — mdlink → /brain/rules/agent-name-contract.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/model-selection-flow.md — mdlink → /brain/rules/env-and-integrations.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — frontmatter → agent-name-contract does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — frontmatter → copilotkit-runtime-route-convention does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — frontmatter → delete-resources-hitl-contract does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — mdlink → /brain/rules/agent-name-contract.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — mdlink → /brain/rules/delete-resources-hitl-contract.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — mdlink → /brain/rules/design-system-convention.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)
⚠ playbooks/research-chat-flow.md — mdlink → /brain/rules/agent-name-contract.md does not resolve in the bundle (not-yet-written knowledge, a typo, or a moved file)

**All pass.**
