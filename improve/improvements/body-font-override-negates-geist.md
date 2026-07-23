---
schemaVersion: 1
id: body-font-override-negates-geist
priority: P3
lens: product
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
---
`layout.tsx:6-15` loads two local Geist fonts via `next/font/local` and exposes them as
`--font-geist-sans` / `--font-geist-mono` on `<body>` ([nextjs-app-shell-convention]
(/brain/rules/nextjs-app-shell-convention.md)). But `globals.css:6` overrides
`body { font-family: Arial, Helvetica, sans-serif; }`, so the Geist fonts are downloaded and
their CSS variables set, yet body text renders in Arial — the fonts are shipped but unused. The
brain flags this as a "known quirk"; it is a silent inconsistency worth resolving one way or the
other: point body at `var(--font-geist-sans)`, or remove the unused Geist loading if Arial is the
deliberate choice. Low impact, trivial fix.
