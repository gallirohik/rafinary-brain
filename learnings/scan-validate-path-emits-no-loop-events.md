---
id: scan-validate-path-emits-no-loop-events
type: Learning
title: The scan/validate path emits no verdict — terminal rulings leave no trace
description: >-
  The validate SOP has no verdict-emit step, so a multi-round scan-QA loop is represented by
  at most incidental events; the terminal ruling that authorizes an artifact is exactly the
  one missing, and zero gate-result events exist despite live gate runs.
pattern: >-
  The scan/validate QA path carries no verdict-emit step in its SOP, so a multi-round
  validation loop leaves at most incidental events: intermediate rulings land by accident,
  terminal rulings go unrecorded, and the deterministic gate runs are witnessed only as
  prose inside a judge's own verification field rather than as independent events.
category: [prism-verdict, gate-result]
evidence_shape:
  - "prism-verdict :: ITERATE · subject names an INTERMEDIATE round of a multi-round scan-QA loop · tier full · method live · major-severity findings — the only scan-path event in the window"
  - "prism-verdict :: NO terminal verdict event exists for that same subject — the ruling that closes the loop and authorizes the artifact is absent while an intermediate one is present"
  - "gate-result :: ZERO events across the entire observed window, although the one enveloped verdict attests live gate runs (exit 0) inside its own verification prose"
  - "category distribution :: 16/17 prism-verdict + 1/17 review-verdict · gate-result, reflex-outcome and distill-refutation all empty across a full build cycle AND a full scan-QA cycle"
proposed_diff_target: .claude/skills/rafa-validate/SKILL.md
proposed_change: >-
  Add an explicit emit step to the validate procedure: one prism-verdict per ROUND —
  including the TERMINAL one — with a round-indexed dedupeKey and the round's real gate
  exit codes as live verification. Extend the existing "prove the machinery first" step to
  assert gate-result sensor liveness: a window in which gates demonstrably ran but no
  gate-result event landed is itself a named health finding.
status: proposed
leverage: { impact: high, effort: low }
---

## Correction (post-pass, same session)

This pass read the repo's checked-out branch (`feat/rafa-rescan`); `.claude/skills/rafa-validate/SKILL.md`
is untouched by the in-flight blueprint update on unmerged branch `chore/update-rafa-0.14.0`
(research-canvas PR #16), so this finding stands as originally written — still fully open.

## The miss class

The build path emits verdicts at task boundaries because its SOP says so, in detail. The
scan/validate path has no equivalent instruction anywhere in its procedure — so emission
there is incidental. The observed window shows the consequence precisely: a multi-round
scan-QA loop is represented by exactly one event, and that event is an *intermediate*
ITERATE. The rounds before it and the terminal ruling after it left no trace.

This inverts the substrate's bias in the worst possible direction. The rulings that matter
most to a self-improvement loop are the ones that **authorize** an artifact — the terminal
PASS. Those are exactly the rulings the observer cannot see on this path. A verdict that
wrongly authorized something leaves no event to learn from; only the iterations that were
already caught are recorded. An observer reading this store would conclude the validate
path is healthy because its failures are visible and its authorizations are not.

## The second half: gates witnessed only as prose

Across the whole window there is not one `gate-result` event, even though the single
enveloped verdict's own verification field attests that deterministic checks ran live and
exited 0. The gate outcomes therefore exist only *inside* a judge's narrative. This defeats
the point of having two planes: the deterministic plane cannot corroborate the judge,
cannot contradict it, and cannot be audited independently of it. Every recorded PASS in
this window ultimately rests on a judge's word.

Emission of `gate-result` is specified as mechanical, which makes its total absence a
sensor-liveness question rather than an authoring one — and nothing in the loop currently
notices a silent sensor. Folding that assertion into the step that already exists to prove
the capture machinery is alive costs one paragraph and closes the blind spot.

## Why this target, and why first

One small addition to a single SOP makes an entire agent plane observable. Until the scan/
validate path emits per round, every future observer pass reasons about it from a sample of
one — and the classes of miss that escape a terminal verdict remain permanently invisible,
including the class recorded separately as `validate-has-no-prior-state-lens`.
