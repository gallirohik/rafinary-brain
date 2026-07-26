---
id: validate-has-no-prior-state-lens
type: Learning
title: Scan QA has no prior-state lens — duplication is structurally invisible
description: >-
  Scan QA validates the produced artifact against the code only; a re-authoring pass that
  mints parallel ids over concepts the store already covers is structurally unreachable to
  the validator, since the artifact's own prior state is not among its declared inputs.
pattern: >-
  Scan QA validates the produced knowledge artifact against the code only; the artifact's
  own PRIOR state is not among its declared inputs. A re-authoring pass that mints parallel
  ids over concepts the store already covers is therefore structurally unreachable for the
  validator — there is no check that can fail on it, so it is not a matter of attention.
category: [prism-verdict, gate-result]
evidence_shape:
  - "prism-verdict :: ITERATE · tier full · method live · multi-round scan-QA loop whose findings are all code-grounded dimensions — the loop iterated only on artifact-vs-code axes"
  - "prism-verdict :: no verdict in the observed window carries a severity finding referencing prior-artifact-state divergence — the class is absent from the ruling vocabulary, not merely unfound"
  - "gate-result :: ZERO events — no deterministic check corroborates a produced artifact against its predecessor state"
proposed_diff_target: .claude/skills/rafa-validate/SKILL.md
proposed_change: >-
  Add the artifact's prior state to Inputs/preconditions, and a prior-state diff check to
  the procedure: enumerate the concepts already present before the pass, and rule any newly
  minted id that names an already-covered concept a BLOCKER-class finding (supersede or
  merge, never mint in parallel). An authoring pass that grows the concept count while
  duplicating existing names fails the net-positive law and must not be able to score its
  way to a PASS.
status: proposed
leverage: { impact: high, effort: low }
---

## Correction / sharpened priority (post-pass, same session)

This pass read the repo's checked-out branch (`feat/rafa-rescan`), which predates an
in-flight blueprint update. On unmerged branch `chore/update-rafa-0.14.0` (research-canvas
PR #16), `.claude/agents/prism.md`'s duty line was already patched to STATE this exact rule
("a REFRESH that minted parallel ids over existing concepts is a BLOCKER-class finding") —
but `.claude/skills/rafa-validate/SKILL.md`, the procedure that duty line points to, was left
untouched by that same update. The card now claims a check the SOP does not implement: a
stated duty with no procedural body behind it is a more actionable version of this finding,
not a weaker one — the rule is documented as already covered, so a reviewer skimming
`prism.md` alone would reasonably believe this class is already caught. It is not. The
authoring-side companion (`authoring-sop-lacks-prior-knowledge-precondition`) IS fixed on that
same branch; this validator-side half is the one gap actually left in the pair.

## The miss class

The validator's independence creed names two things to review: the produced artifact and
the code as ground truth. That pairing is what makes it strong at fidelity — every claim is
re-derived from the code. It is also exactly what makes this class invisible: **duplication
of existing knowledge is not a code-grounded property.** Every individual note in a
re-authored store can cite correctly, describe the code truthfully, and score well, while
the store as a whole has silently doubled — parallel ids covering concepts that already had
owners.

Because the prior state is not an input, no procedure step, no severity rule and no score
dimension can express the failure. The verdict vocabulary observed in the window bears this
out: rulings iterate on coverage balance, essence and claim truth, and never on divergence
from the artifact's predecessor. This is a structural blind spot, not an agent lapse — the
strongest kind of finding for a card diff, because no amount of diligence closes it.

## Why a validator check, not only an authoring precondition

The companion entry proposes a hydrate-before-author precondition on the producing side.
That is necessary but not sufficient: a precondition inside a procedure is a step that can
be skipped, and a skipped step with no gate behind it is undetectable. Defense in depth is
the whole reason the producer and the validator are separate agents. A miss class that
survives both a producer's procedure and a validator's full-tier live pass, and is caught
only downstream, is by definition a gap in the validator's lens — the producer can always
err; the validator existing to catch producer error is the point.

## Cost and shape of the fix

The check is cheap and mechanical in character: compare the set of concept ids/names before
and after, and flag new ids that name existing concepts. It needs no new tooling category —
it slots beside the existing adversarial probes, and its severity is naturally blocker-class
under the already-stated law that a misleading store is worse than none. Two duplicate
descriptions of one concept are precisely that: whichever the reader finds first may be the
stale one.
