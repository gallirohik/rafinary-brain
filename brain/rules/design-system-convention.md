---
schemaVersion: 1
id: design-system-convention
type: convention
domain: components
title: shadcn/ui (new-york) primitives + cn() + hardcoded brand accents
summary: UI is shadcn "new-york" Radix primitives in components/ui with CSS-variable tokens and the cn() merge helper; feature components layer on top but hardcode the #6766FC/#0E103D brand colors inline rather than as tokens
links: [nextjs-app-shell-convention]
cites:
  - components.json:3 :: new-york
  - src/components/ui/button.tsx:7 :: buttonVariants
  - src/lib/utils.ts:4 :: cn
  - tailwind.config.ts:13 :: hsl(var(--background))
  - src/app/globals.css:41 :: --radius
  - src/components/AddResourceDialog.tsx:35 :: #6766FC
  - src/app/Main.tsx:21 :: #0E103D
  - src/components/ResearchCanvas.tsx:1 :: "use client"
---
The component layer has two tiers.

**Tier 1 — `src/components/ui/*` — generated shadcn primitives.** Style is `new-york`,
`rsc: true`, base color `neutral`, CSS variables on (`components.json`). Each primitive is a
Radix wrapper (`button`, `card`, `dialog`, `input`, `select`, `textarea`) built with
`class-variance-authority` for variants (`button.tsx:7` `buttonVariants`) and merged with
`cn()` (`src/lib/utils.ts:4` = `twMerge(clsx(...))`). Colors are semantic tokens
(`bg-primary`, `bg-background`, `text-destructive`) resolved through
`hsl(var(--token))` in `tailwind.config.ts:13+` and defined in `globals.css` `:root`
(`--radius: 0.5rem` at `:41`). To add a primitive, generate via the shadcn CLI (aliases in
`components.json`), don't hand-roll.

**Tier 2 — `src/components/*` — feature components** (7 files). `ResearchCanvas`,
`Resources`, `Progress`, `FactCheck`, `AddResourceDialog`, `EditResourceDialog`,
`ModelSelector`. They compose Tier-1 primitives and use `lucide-react` icons.

**Only two of the seven declare `"use client"`** — `ResearchCanvas.tsx:1` and
`ModelSelector.tsx:1`. The other five (`Resources`, `Progress`, `FactCheck`,
`AddResourceDialog`, `EditResourceDialog`) carry no directive: they are *already* client
components because every import chain into them starts at `page.tsx`, which opens the
boundary ([nextjs-app-shell-convention](/brain/rules/nextjs-app-shell-convention.md)). So
"no `"use client"` here" does NOT mean "this is a Server Component" in this repo — it means
the boundary was opened upstream. Don't infer RSC-ness from the absence of the directive,
and don't add server-only code to these files.

**Non-exemplar to know (don't blindly copy):** the brand accent colors are **hardcoded hex**
inline — the CopilotKit purple `#6766FC` (`AddResourceDialog.tsx:35`, and buttons throughout
`ResearchCanvas`) and the header navy `#0E103D` (`Main.tsx:21`) — instead of being added as
`--primary`/theme tokens. The design-token system exists but these accents bypass it, so a
theme change won't touch them. If you restyle, grep the raw hex, don't just edit tokens.
