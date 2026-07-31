---
schemaVersion: 1
id: lgc-deployment-url-ssrf-asymmetry
priority: P2
category: security
status: fixed
title: The lgcDeploymentUrl SSRF guard checks host strings only — no DNS resolution, unlike the Python side
summary: isSafeDeploymentUrl blocked private hostnames by literal/regex pattern but never resolved DNS, so a public hostname pointing at a private/metadata IP passed and the proxy forwarded langsmithApiKey to it — now fixed; the guard resolves DNS and checks every resolved address, mirroring download.py
fix: Resolve the hostname and reject when any resolved address is private/loopback/link-local, mirroring download.py's _is_safe_url (~15 min)
leverage: { impact: medium, effort: medium }
blast_radius: [agent-bridge, external-integrations, security]
cites:
  - src/app/api/copilotkit/route.ts:59 :: isSafeDeploymentUrl
  - src/app/api/copilotkit/route.ts:74 :: dns.lookup
  - src/app/api/copilotkit/route.ts:29 :: isPrivateIPv4
  - src/app/api/copilotkit/route.ts:43 :: isPrivateIPv6
  - agents/python/src/lib/download.py:41 :: getaddrinfo
found: 2026-07-20
fixed: 2026-07-26
type: Improvement
description: "isSafeDeploymentUrl blocked private hostnames by literal/regex pattern but never resolved DNS, so a public hostname pointing at a private/metadata IP passed and the proxy forwarded langsmithApiKey to it — now fixed; the guard resolves DNS and checks every resolved address, mirroring download.py"
timestamp: 2026-07-20
tags: [security, P2]
---
`isSafeDeploymentUrl` is the guard that keeps the runtime proxy — which carries
`langsmithApiKey` (`route.ts:19`) — from being redirected at internal hosts
([copilotkit-runtime-route-convention](/brain/rules/copilotkit-runtime-route-convention.md)).
It used to reject private targets purely by inspecting the URL's `hostname` string
(`localhost`, `127.`, `10.`, `169.254.`, …) and never resolved DNS, so a public hostname
(`evil.example`) whose A record pointed at `169.254.169.254` or an internal `10.x` address
passed the check and the server then forwarded the LangSmith key to that address.

The Python resource downloader guarded the same SSRF class **correctly** — `_is_safe_url`
resolves via `socket.getaddrinfo` (`download.py:41`) and rejects if *any* resolved IP is
private/loopback/link-local. The asymmetry was the finding: the proxy's guard was weaker than
the backend's for the identical threat, on an opt-in path (`?lgcDeploymentUrl=`), hence P2.

## Resolution (2026-07-26)

The proxy guard now matches the Python pattern. `isSafeDeploymentUrl` (`route.ts:59-84`) is
`async` and:
1. rejects non-`https:` and `localhost`/`*.local` up front (`:66-69`);
2. resolves the hostname with `dns.lookup(hostname, { all: true, verbatim: true })`
   (`route.ts:74`), returning false if resolution throws or yields zero addresses;
3. requires **every** resolved address to be public, via dedicated `isPrivateIPv4`
   (`route.ts:29`) and `isPrivateIPv6` (`route.ts:43`) predicates — covering `0.x`, `10.x`,
   `127.x`, `169.254.x`, `172.16–31.x`, `192.168.x`, multicast/reserved `≥224`, and on the
   v6 side `::1`, `::`, IPv4-mapped `::ffff:`, `fe80::/10`, `fc00::/7` and `ff…` multicast.

`POST` awaits it and 400s on rejection (`route.ts:93-101`). Verified against the current
source this pass. The guard is now stricter than the original finding asked for (the IPv6
and multicast cases were not in the proposed fix); don't "simplify" it back to a hostname
string check — that is the exact regression this row records.

<!-- okf:citations:start (generated — the frontmatter `cites:` DSL is the source of truth; do not hand-edit) -->

# Citations

[1] [src/app/api/copilotkit/route.ts:59](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L59) — `isSafeDeploymentUrl`
[2] [src/app/api/copilotkit/route.ts:74](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L74) — `dns.lookup`
[3] [src/app/api/copilotkit/route.ts:29](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L29) — `isPrivateIPv4`
[4] [src/app/api/copilotkit/route.ts:43](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/src/app/api/copilotkit/route.ts#L43) — `isPrivateIPv6`
[5] [agents/python/src/lib/download.py:41](https://github.com/gallirohik/research-canvas/blob/cdd463ba519f6da63d04b45d31da5f4f254d0790/agents/python/src/lib/download.py#L41) — `getaddrinfo`

<!-- okf:citations:end -->
