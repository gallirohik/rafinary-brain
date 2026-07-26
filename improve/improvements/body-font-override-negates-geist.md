---
schemaVersion: 1
id: body-font-override-negates-geist
priority: P3
category: product
status: open
title: globals.css hardcodes Arial on body, negating the two Geist fonts loaded in layout
summary: layout.tsx loads Geist Sans + Geist Mono via next/font/local and wires the CSS variables, but globals.css sets body font-family to Arial, so the loaded fonts never apply to body text — dead weight plus off-brand typography
fix: Set body font-family to var(--font-geist-sans) (or drop the Geist loading if Arial is intended) (~5 min)
leverage: { impact: low, effort: low }
blast_radius: [routing-app-shell, components]
cites:
  - src/app/globals.css:6 :: Arial
  - src/app/layout.tsx:6 :: localFont
found: 2026-07-20
type: Improvement
description: "layout.tsx loads Geist Sans + Geist Mono via next/font/local and wires the CSS variables, but globals.css sets body font-family to Arial, so the loaded fonts never apply to body text — dead weight plus off-brand typography"
timestamp: 2026-07-20
---
`layout.tsx:6-15` loads two local Geist fonts via `next/font/local` and exposes them as
`--font-geist-sans` / `--font-geist-mono` on `<body>` ([nextjs-app-shell-convention]
(/brain/rules/nextjs-app-shell-convention.md)). But `globals.css:6` overrides
`body { font-family: Arial, Helvetica, sans-serif; }`, so the Geist fonts are downloaded and
their CSS variables set, yet body text renders in Arial — the fonts are shipped but unused. The
brain flags this as a "known quirk"; it is a silent inconsistency worth resolving one way or the
other: point body at `var(--font-geist-sans)`, or remove the unused Geist loading if Arial is the
deliberate choice. Low impact, trivial fix.

Still present as of the 2026-07-27 refresh: `globals.css:5-7` is unchanged, and `layout.tsx`
still loads both fonts and applies `${geistSans.variable} ${geistMono.variable}` to `<body>`
(`layout.tsx:30`). Note the `body` rule sits OUTSIDE `@layer base` (`globals.css:15`), so it
wins on specificity/order against the token layer — moving it inside the layer is not the fix.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/globals.css:6](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/globals.css#L6) — `Arial`
[2] [src/app/layout.tsx:6](https://github.com/gallirohik/research-canvas/blob/c31971e8a2b5a4992aee13917704e47e492369d7/src/app/layout.tsx#L6) — `localFont`

<!-- okf:citations:end -->
