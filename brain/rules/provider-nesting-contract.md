---
schemaVersion: 1
id: provider-nesting-contract
type: contract
domain: routing-app-shell
title: Provider nesting order — ModelSelectorProvider outside CopilotKit outside coagent hooks
summary: useModelSelectorContext throws outside ModelSelectorProvider, and useCoAgent needs CopilotKit above it; the app-shell wrapping order in page.tsx is load-bearing and fails loudly if inverted
links: [nextjs-app-shell-convention, copilotkit-runtime-route-convention, agent-name-contract]
anchor: none  # composition/ordering contract — provider nesting, not a single token
failure: loud
cites:
  - src/app/page.tsx:13 :: ModelSelectorProvider
  - src/app/page.tsx:21 :: useModelSelectorContext
  - src/app/page.tsx:28 :: CopilotKit
  - src/app/page.tsx:36 :: Main
  - src/lib/model-selector-provider.tsx:66 :: throw new Error
description: "useModelSelectorContext throws outside ModelSelectorProvider, and useCoAgent needs CopilotKit above it; the app-shell wrapping order in page.tsx is load-bearing and fails loudly if inverted"
tags: [routing-app-shell]
timestamp: 2026-07-26T22:44:42.840Z
---
The app shell is a fixed stack of React context providers, and the nesting order is a
contract — invert it and the app crashes on render (loud, not silent).

The order, outermost first (`src/app/page.tsx`):
1. `<ModelSelectorProvider>` (`page.tsx:13`) — supplies `{ model, agent, lgcDeploymentUrl }`.
2. `<Home>` and `<ModelSelector>` as children. `Home` calls
   `useModelSelectorContext()` (`page.tsx:21`); `ModelSelector` calls it too. That hook
   **throws** `"must be used within a ModelSelectorProvider"` if no provider is above it
   (`model-selector-provider.tsx:66`).
3. `<CopilotKit ...>` (`page.tsx:28`), configured from the model-selector context, wrapping
   `<Main/>` (`page.tsx:36`).
4. `useCoAgent` / `useCoAgentStateRender` / `useCopilotAction` inside `Main` and
   `ResearchCanvas` require a `CopilotKit` provider above them — they read its runtime.

So the invariant chain is: **ModelSelectorProvider → (context consumers) → CopilotKit →
(coagent hooks)**. `agent` is computed in the outer context and passed into `CopilotKit`, so
the outer provider must resolve before the inner one is configured. Moving a coagent hook
above `CopilotKit`, or a context consumer above `ModelSelectorProvider`, breaks at runtime.
See [the app-shell convention](/brain/rules/nextjs-app-shell-convention.md). `anchor: none`
because provider nesting is a composition contract, not a greppable token.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/page.tsx:13](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/page.tsx#L13) — `ModelSelectorProvider`
[2] [src/app/page.tsx:21](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/page.tsx#L21) — `useModelSelectorContext`
[3] [src/app/page.tsx:28](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/page.tsx#L28) — `CopilotKit`
[4] [src/app/page.tsx:36](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/page.tsx#L36) — `Main`
[5] [src/lib/model-selector-provider.tsx:66](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/lib/model-selector-provider.tsx#L66) — `throw new Error`

<!-- okf:citations:end -->
