# Learnings ledger — sage (L5: the system → itself)

Proposed diffs to OUR agent cards and SOPs, cited to loop-event SHAPES only.
Asset-free and person-free by construction: no customer code, no repo facts, no dev
observations. **Nothing here self-applies** — each entry is a proposal for human/MR review;
acceptance lands separately as a versioned card/SOP edit.

- **Pass 1** — first observer pass on this repo (no prior ledger).
- **Window** — 17 loop events, spanning one full build cycle and one full scan-QA cycle.
- **Substrate health** — 1/17 events fully enveloped; `gate-result`, `reflex-outcome` and
  `distill-refutation` are empty categories across the entire window.

## Correction (post-pass, same session)

This pass read the repo at its checked-out branch (`feat/rafa-rescan`), which predates an
in-flight blueprint update sitting on unmerged branch `chore/update-rafa-0.14.0` (research-canvas
PR #16) — that update already patches `.claude/skills/rafa-scan/SKILL.md` (a mandatory Step 0:
`rafa pull --full`, a founding-vs-refresh mode branch, tombstone-not-delete) and
`.claude/agents/prism.md`'s duty line (states the parallel-id BLOCKER-class rule), both
addressing exactly the incident this window's evidence comes from. Entry #3 below is
**superseded** by that pending PR. Entries #1 and #2 target `.claude/skills/rafa-validate/SKILL.md`,
which that same update left untouched — those remain fully open, and #2 is now the higher-priority
half in practice: `prism.md` now STATES a duplicate-id check its own procedure file doesn't
implement, so the gap reads as already-closed to anyone skimming the card.

## Top leverage (impact x ease — apply in this order)

1. **[validate-has-no-prior-state-lens](validate-has-no-prior-state-lens.md)** —
   scan QA reviews artifact-vs-code only; re-authoring over existing concepts is structurally
   unreachable, so no check can fail on it. `prism.md` already claims this check exists
   (pending PR #16) — the SOP behind that claim does not yet.
2. **[scan-validate-path-emits-no-loop-events](scan-validate-path-emits-no-loop-events.md)** —
   the validate SOP has no verdict-emit step, so an entire agent plane is unobservable and
   its *terminal* rulings are the ones missing. One SOP addition; unblocks every future pass.
3. ~~**[authoring-sop-lacks-prior-knowledge-precondition](authoring-sop-lacks-prior-knowledge-precondition.md)**~~ —
   superseded: already fixed on unmerged branch `chore/update-rafa-0.14.0` (PR #16), independent
   of this pass.
4. **[envelope-null-events-skew-rate-analysis](envelope-null-events-skew-rate-analysis.md)** —
   read-side only: the observer must not average legacy null-envelope events into method/tier
   splits, nor count duplicate-shaped emits twice.

## By status

| status | count |
| --- | --- |
| proposed | 3 |
| accepted | 0 |
| rejected | 0 |
| superseded | 1 |

## By proposed diff target

| target | count | entries |
| --- | --- | --- |
| `.claude/skills/rafa-validate/SKILL.md` | 2 | emit-per-round + gate-result liveness; prior-state diff check |
| `.claude/skills/rafa-scan/SKILL.md` | 1 (superseded) | hydrate-before-author precondition + refresh mode — already landed independently on PR #16 |
| `.claude/skills/rafa-sage/SKILL.md` | 1 | envelope-coverage + duplicate-collapse before any rate |

## By evidence category

| category | entries drawing on it |
| --- | --- |
| `prism-verdict` | 4 |
| `gate-result` | 2 (both citing its total ABSENCE as the evidence) |
| `review-verdict` | 1 |
| `reflex-outcome` | 0 — no events exist in this category |
| `distill-refutation` | 0 — no events exist in this category |

## Explicitly NOT proposed (checked, already closed)

- **Emit-side idempotency and actor envelope.** The duplicate-shaped and null-envelope events
  in this window are pre-wave-5 legacy. The current build SOP already mandates the atomic
  task-boundary beat with a client-stable dedupeKey, the actor envelope and the verification
  entry, and records the non-idempotent-emit learning as closed; `gate-result` is specified as
  mechanically CLI-stamped. Only the READ-side residual is proposed (entry 4).
