---
id: envelope-null-events-skew-rate-analysis
type: Learning
title: Unfiltered envelope-null and duplicate-shaped events skew observer rates
description: >-
  The observer's SOP requires splitting outcome rates by verification method, actor envelope
  and tier, but the store mixes pre-envelope events (all three null) with enveloped ones and
  duplicates some rulings with no dedupeKey — a naive rate averages legacy with current and
  double-counts outcomes.
pattern: >-
  The observer's own SOP requires splitting outcome rates by verification method, actor
  envelope and tier, but the store mixes pre-envelope events (all three dimensions null) with
  enveloped ones, and pre-idempotency events duplicate the same ruling. A naive rate silently
  averages legacy with current and double-counts outcomes — the observer can draw a confident
  conclusion from a substrate that cannot support it.
category: [prism-verdict, review-verdict]
evidence_shape:
  - "envelope coverage :: 16/17 events carry actorMeta, verification and tier all null · 1/17 fully enveloped — the mandated split dimensions are unavailable for ~94% of the window"
  - "prism-verdict :: 5 distinct build-task subjects each show a consecutive SAME-outcome PASS pair seconds apart, dedupeKey null — 10/17 events duplicate-shaped, inflating PASS count ~2x"
  - "prism-verdict :: one subject shows ITERATE then PASS ~5s apart with no verification method recorded — too short an interval for an executed re-check to have produced the flip"
proposed_diff_target: .claude/skills/rafa-sage/SKILL.md
proposed_change: >-
  Require every rate the observer reports to state its envelope-coverage fraction, and to
  EXCLUDE envelope-null events from method/tier/actor splits rather than averaging them in.
  Require a duplicate-shape screen before any rate is computed: same subject + same outcome +
  null dedupeKey within a short window collapses to one observation. Legacy pre-envelope
  emits must never be readable as current health.
status: proposed
leverage: { impact: medium, effort: low }
---

## Scope note for the reviewer — this is a READ-side learning only

The write-side of this class is already CLOSED and must not be re-fixed. The current build
SOP mandates the atomic task-boundary beat carrying the actor envelope, the verification
entry and a client-stable dedupeKey, and explicitly records the non-idempotent-emit learning
as closed; gate-result emission is likewise specified as mechanical and CLI-stamped. New
emits cannot reproduce these shapes. Proposing emit-side changes here would be re-litigating
a settled fix.

What is *not* closed is the reading side. The store is permanent and now contains both eras.
Nothing in the observer's procedure distinguishes them, yet the creed instructs it to split
every rate by exactly the three dimensions the older era lacks. That instruction, applied
literally to this window, would compute splits over a single event while presenting them as
the window's behaviour.

## The two distortions, separately

**Dimension nullity.** With ~94% of events carrying no verification method, any claim of the
form "the PASS rate holds only on static rulings" is unsupportable — not false, but
unfounded. The honest output is a coverage fraction alongside the rate, so a thin substrate
reads as thin rather than as evidence.

**Duplicate inflation.** Five separate subjects each contributed two identical consecutive
outcomes with no dedupe key. An observer counting outcomes naively sees roughly twice the
successes that occurred. Since outcome counts are the input to every recurrence threshold,
this distortion propagates: it can push a non-pattern over a threshold, or mask a real one
by diluting the failure share. A collapse screen is a few lines of procedure and removes the
whole hazard.

## Why it is worth a card diff at all

The observer exists to keep other agents honest about evidence. A procedure that mandates
splits its own substrate may not support is the same failure mode it is built to catch,
turned inward. The fix is cheap, self-applied at read time, and permanently correct as the
store's composition shifts.
