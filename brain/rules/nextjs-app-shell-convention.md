---
schemaVersion: 1
id: nextjs-app-shell-convention
type: convention
domain: routing-app-shell
title: Next.js App Router shell — a single client-rendered page under a thin RSC layout
summary: The whole app is one route; layout.tsx is the only server component (fonts + metadata), and everything from page.tsx down is "use client" because the coagent hooks need the browser
links: [provider-nesting-contract, copilotkit-runtime-route-convention, design-system-convention]
cites:
  - src/app/layout.tsx:22 :: RootLayout
  - src/app/layout.tsx:6 :: localFont
  - src/app/page.tsx:1 :: "use client"
  - src/app/Main.tsx:8 :: Main
  - src/app/globals.css:15 :: @layer base
---
This is a **single-route** Next.js 15 App Router app — there are no nested route segments,
no `[params]`, no additional `page.tsx` files. Orientation:

- **`layout.tsx` is the only real Server Component.** It sets `metadata` (`layout.tsx:17-20`
  — now the real product title, no longer the create-next-app scaffold), loads the two
  local Geist fonts via `next/font/local` (`layout.tsx:6-15`), imports global CSS +
  `@copilotkit/react-ui/styles.css`, and renders `{children}` inside `<body>`
  (`layout.tsx:22-35`). Keep it server-side; don't add `"use client"` here.
- **`page.tsx` opens the client boundary** with `"use client"` at the top (`page.tsx:1`).
  Everything below it — `Main`, `ResearchCanvas`, the dialogs, the model selector — is
  client-rendered. This is deliberate and unavoidable: CopilotKit's `useCoAgent`/
  `useCopilotAction` hooks and the model-selector context need browser state (URL params,
  live streaming). Do **not** try to make these RSC. Corollary: most files under
  `src/components/` carry no `"use client"` directive yet are still client components —
  the boundary was opened here, once. See
  [the design-system convention](/brain/rules/design-system-convention.md).
- **`page.tsx` also owns provider composition** — see
  [the provider nesting contract](/brain/rules/provider-nesting-contract.md). `Main.tsx` is
  the layout frame (header + split pane: `ResearchCanvas` left, `CopilotChat` right), seeds
  the coagent `initialState` from the shared `createInitialAgentState` factory
  (`Main.tsx:10-13`), and registers chat suggestions via `useCopilotChatSuggestions`
  (`Main.tsx:15-17`).
- **Path aliases**: `@/` → `src/` (see `components.json` aliases: `@/components`, `@/lib`,
  `@/components/ui`). Import via aliases, not relative paths.
- **Global styles**: Tailwind layers + shadcn CSS variables live in
  `src/app/globals.css` (`:15` `@layer base` holds the design tokens). Body font is
  overridden to Arial there despite the Geist font vars being wired in layout — a known
  quirk, not a convention to copy.
