---
id: authoring-sop-lacks-prior-knowledge-precondition
type: Learning
title: Scan SOP had no hydrate-before-author precondition (superseded)
description: >-
  The founding-authoring SOP was written for a first pass over an empty store with no
  precondition to check for pre-existing shared knowledge, so a repeat run silently minted
  parallel concepts; already fixed independently on a pending blueprint update (see body).
pattern: >-
  The founding-authoring SOP is written throughout for a FIRST pass over an empty store and
  declares no precondition to hydrate pre-existing shared knowledge before authoring. Re-running
  it against a store that already holds knowledge is therefore fully procedurally valid, and
  produces parallel concepts silently — the SOP has no mode distinction between founding and
  refresh, so there is nothing to skip and nothing to violate.
category: [prism-verdict]
evidence_shape:
  - "prism-verdict :: ITERATE · tier full · method live · multi-round scan-QA loop following an authoring pass — remediation cost paid downstream, in rounds, rather than prevented at the precondition"
  - "prism-verdict :: the authoring path's rulings in the window are all post-hoc quality judgements; no event shape corresponds to a pre-authoring precondition check"
  - "gate-result :: ZERO events — no deterministic gate stands between an authoring pass and an already-populated store"
proposed_diff_target: .claude/skills/rafa-scan/SKILL.md
proposed_change: >-
  Promote hydrate-before-author to a hard PRECONDITION ahead of the procedure, with an
  explicit two-mode branch: pull existing shared knowledge first, then EMPTY store → founding
  pass; NON-EMPTY store → refresh mode, which supersedes or merges existing ids and never
  mints parallel ones. State it as a gate with a named failure rather than a numbered step
  inside the procedure, so that skipping it is a visible violation instead of an ordinary
  ordering choice.
status: superseded
leverage: { impact: high, effort: low }
---

## Correction (post-pass, same session)

This pass ran against the repo's checked-out branch at the time (`feat/rafa-rescan`), which
predates an in-flight blueprint update. On branch `chore/update-rafa-0.14.0` (unmerged,
research-canvas PR #16), `.claude/skills/rafa-scan/SKILL.md` already carries almost exactly
this proposal: a numbered **Step 0** ("mandatory and mechanical"), `rafa pull --full` before
any scan step, an explicit founding-vs-refresh mode branch keyed on whether the platform
serves any knowledge, stable ids, tombstone-not-delete, and an anti-pattern entry naming
this exact incident ("28 duplicates against 14 existing notes"). The fix independently
predates this pass — status set to `superseded`, not `accepted`, since no reviewer accepted
it off this ledger. The companion validator-side gap below is NOT covered by that update
(`rafa-validate/SKILL.md` was untouched) and remains fully open — see
`validate-has-no-prior-state-lens`, which this entry's own "why a validator check" section
already argued is the higher-priority half.

## The miss class

Read as a whole, the authoring SOP assumes an empty destination. Its framing is a founding
contribution, its prime directive is breadth-before-depth over a whole repo, and its
acceptance criteria measure the produced set on its own terms. Nowhere does it require the
agent to first learn what the shared store already contains. There is no precondition block
naming prior knowledge as an input, and no branch distinguishing a first pass from a repeat
pass.

The consequence is that a repeat run is not a deviation — it is the SOP executed correctly.
An agent following the procedure verbatim over a populated store will produce a fresh,
internally coherent, correctly cited set of concepts that happen to duplicate ones already
owned. The failure is authored into the blueprint, not into the agent's behaviour.

## Why "make it a gate", not "make it more prominent"

The tempting fix is to move the instruction earlier or bold it. That does not change the
class. A numbered step is an ordering; an agent under context pressure that starts from the
prime directive can rationally begin authoring without it, and nothing downstream registers
the omission. A precondition with a named failure changes the shape: the pass either
establishes which mode it is in, or it has not started. Mode selection also gives the
refresh path somewhere to live — today the SOP describes only how to create, never how to
supersede, so even an agent that *does* check has no procedure for the non-empty case.

## Relationship to the validator entry

This is the producer half of a two-sided gap; `validate-has-no-prior-state-lens` is the
validator half. Applying only this one leaves a skippable step with no gate behind it;
applying only that one lets the error be made every time and caught late, at iteration cost.
The pair is what makes the class recoverable: prevented at authoring, and caught if
prevention is skipped. Reviewers should weigh them together, and if only one lands first,
prefer the validator half — a gate that catches beats a step that reminds.
