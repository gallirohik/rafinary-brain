---
schemaVersion: 1
id: next-transitive-postcss-sharp-cves
priority: P1
category: security
status: open
title: postcss@8.4.31 and sharp@0.34.5 pulled in under next carry 4 advisories (3 high)
summary: Two packages resolved transitively through next@15.5.15 are vulnerable — postcss 8.4.31 (arbitrary file read, path traversal in source-map auto-loading, XSS in stringify output) and sharp 0.34.5 (inherited libvips CVEs); both are fixable now with a pnpm override, without waiting on the next 15 -> 16 major
fix: Add pnpm.overrides for `postcss: ">=8.5.18"` and `sharp: ">=0.35.0"`, reinstall, re-run `npx @rafinery/cli audit` (~15 min)
leverage: { impact: medium, effort: low }
blast_radius: [build-tooling, security]
cites:
  - pnpm-lock.yaml:3045 :: postcss@8.4.31
  - pnpm-lock.yaml:3349 :: sharp@0.34.5
  - package.json:39 :: postcss
  - postcss.config.mjs:4 :: tailwindcss
found: 2026-07-27
type: Improvement
description: "Two packages resolved transitively through next@15.5.15 are vulnerable — postcss 8.4.31 (arbitrary file read, path traversal in source-map auto-loading, XSS in stringify output) and sharp 0.34.5 (inherited libvips CVEs); both are fixable now with a pnpm override, without waiting on the next 15 -> 16 major"
timestamp: 2026-07-27
tags: [security, P1]
---
Source: `npx @rafinery/cli audit --json` (`rafa.audit/v1`, run 2026-07-26), dependency tier.
Machine-sourced from the envelope.

Both packages are **transitive under `next`** (`chain.direct: false`; paths
`next@15.5.15 -> postcss@8.4.31` and `next@15.5.15 -> sharp@0.34.5`). Neither is `dev:true`.
Priority is the mechanical map on the highest severity present: **high -> P1**. Grouped into
one row because they share one fix action (a `pnpm.overrides` block, or the parent `next`
bump).

| package | severity | advisory | fixedIn |
| --- | --- | --- | --- |
| postcss 8.4.31 | high | GHSA-6g55-p… — arbitrary file read / information disclosure via attacker-controlled input | 8.5.12 |
| postcss 8.4.31 | high | GHSA-r28c-9… — path traversal in previous-source-map auto-loading (`sourceMap`) | 8.5.18 |
| postcss 8.4.31 | moderate | GHSA-qx2v-q… — XSS via unescaped `</style>` in CSS stringify output | 8.5.10 |
| sharp 0.34.5 | high | GHSA-f88m-g… — inherited libvips vulnerabilities (CVE-2026-33327, CVE-2026-3…) | 0.35.0 |

`>= 8.5.18` covers all three postcss advisories; `>= 0.35.0` covers sharp.

## Reachability — brain-grounded annotation (priority-neutral)

- **postcss: build-time only.** It runs in the CSS pipeline (`postcss.config.mjs:4` loads
  `tailwindcss`) over first-party stylesheets — `src/app/globals.css` and Tailwind's own
  output ([build-tooling-convention](/brain/rules/build-tooling-convention.md)). All three
  advisories require attacker-controlled CSS or source-map input; this repo compiles no
  untrusted CSS, so the practical exposure is a malicious dependency's stylesheet at build
  time, not a request-path vector. Note the repo also declares `postcss: "^8"` as a devDep
  (`package.json:39`) — the *vulnerable* copy at 8.4.31 is the one next pins, so bumping the
  devDep range alone will not clear these; the override is what moves the resolved version.
- **sharp: not exercised at runtime.** sharp is next's image-optimization backend, and no
  `next/image` import exists anywhere in `src/`. The libvips CVEs need attacker-supplied image
  bytes to reach the decoder; nothing in this app feeds it any.

Both annotations argue the *practical* risk is well below the CVSS headline. Neither
downgrades the row — annotate, never downgrade. The reason to take it anyway is that it is
genuinely cheap: an overrides block and a reinstall, no code change, no API surface moved.

## Sequencing with the major

If `next-dependency-cves` (the 15 -> 16 major) is scheduled soon, upgrading next pulls fresh
postcss and sharp on its own and this row closes for free — do not do both. If the major is
deferred, take the override now: it retires 4 of the 28 findings for ~15 minutes of work.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [pnpm-lock.yaml:3045](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/pnpm-lock.yaml#L3045) — `postcss@8.4.31`
[2] [pnpm-lock.yaml:3349](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/pnpm-lock.yaml#L3349) — `sharp@0.34.5`
[3] [package.json:39](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/package.json#L39) — `postcss`
[4] [postcss.config.mjs:4](https://github.com/gallirohik/research-canvas/blob/0c96b3c1289772846eae57f8768be579cc7d8fe4/postcss.config.mjs#L4) — `tailwindcss`

<!-- okf:citations:end -->
