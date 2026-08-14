# ADR-0003: Four-tier risk classification over a continuous score, with non-accusatory language as a design requirement

**Status:** Accepted (2026-05-21; reconstructed 2026-08-13)

## Context

The auditor's earliest recorded form (`prompts/v3-auditor.md`, scaffolded
2026-03-23) already classified each reference into one of four risk levels —
**Low / Moderate / Elevated / High** — rather than emitting a single
continuous confidence number per reference. `prompts/v4-auditor.md`
(committed 2026-05-21) relabeled the bottom tier **Defensible** and gave the
scheme the letter shorthand (D/M/E/H) that the auditor still uses, tying it
to the aggregate scoring formula documented in `docs/heuristics.md`
(`Score = 100 − H×12 − E×5 − M×2`, floored at 0, with tier weights unchanged
since v4 relative to each other). The per-reference tier is the unit
editorial judgment acts on; the aggregate score is a summary of tiers, not
the other way around.

Heuristic 10 (journal legitimacy / predatory-venue flagging, shipped in v6
via PR [#44](../../pull/44), 2026-06-19) made a second, related requirement
explicit: classification language must be factual and non-accusatory.
`roadmap/v4-features.md` records the resulting rule directly — *"'Predatory'
is not used as a determination; 'potentially predatory' or 'unverified
venue' at most"* — and the same constraint is echoed in the README's H10
description ("never 'predatory' as a verdict"). This generalizes a
constraint that was implicit from the start: the auditor's job is to
surface verifiable anomalies for an editor to judge, not to hand down a
finding of misconduct itself.

## Decision

Classify every reference into one of four discrete risk tiers — Defensible,
Moderate, Elevated, High — instead of a single continuous per-reference
confidence score, and aggregate those tiers into the headline 0–100 score
via fixed per-tier weights. Require classification language to stay
factual and descriptive rather than accusatory at every tier, most
concretely enforced on Heuristic 10: never "predatory" as a verdict, at
most "potentially predatory" or "unverified venue."

## Alternatives

**Recorded at the time:**
- No alternative framing to the four-tier scheme itself is recorded in the
  repo history — the tiered classification is present from the earliest
  captured prompt (v3) onward, so no "before" state exists to contrast
  against in-repo.
- For H10 specifically, the recorded alternative was flagging any journal
  absent from the primary indexes (DOAJ, PubMed/MEDLINE, Scopus, Web of
  Science) as an outright verdict. Rejected in favor of the hybrid
  whitelist-plus-community-list design actually shipped, which treats
  absence from primary indexes plus corroboration from secondary,
  non-authoritative lists (Beall's archived list, Stop Predatory Journals)
  as sufficient only for the softer "Elevated"/"unverified venue" framing,
  never a standalone accusation.

**Retrospective — not considered at the time:**
- **A continuous 0–100 (or 0–1) per-reference confidence score** instead of
  four discrete tiers, with the aggregate simply averaging or summing raw
  confidences. *Worse fit*: a continuous per-reference score invites false
  precision the underlying evidence doesn't support — "this citation is
  73% likely fabricated" implies a calibration the heuristics (pattern
  matches against Crossref/PubMed/publisher metadata) can't actually back
  up, and it's easier to game or dispute at the margins than a discrete
  tier with a stated rationale. It would also have made the exact defect
  issue [#43](../../issues/43) found — the aggregate formula structurally
  punishing clean articles — harder to spot, since a continuous score
  smooths over the same distortion a coarse tier table exposes plainly.
- **A binary flagged/not-flagged classification** (no gradation at all).
  *Worse fit*: collapses the real editorial difference between "minor,
  inconclusive concern" (Moderate) and "strong evidence of fabrication"
  (High) into a single bucket, which would force every flag through the
  same escalation path regardless of severity — directly at odds with the
  graduated COPE-alignment mapping (`docs/heuristics.md`) that routes
  High-tier findings to author query / editorial investigation and treats
  Elevated-tier venue findings as no dedicated flowchart, editorial
  judgment only.

## Consequences

- Every reference gets an auditable, discrete tier with a stated rationale
  instead of an opaque number, which is what makes the per-tier weight
  table (and therefore the headline score) legible and debuggable — it's
  what let issue #43 pin the scoring-formula defect to a specific term
  (`D × 3`) rather than a vague "scores feel off."
- The non-accusatory language requirement means the auditor's output is
  necessarily conservative in its wording even on strong findings — an
  editor reading a High-tier H10 flag still has to exercise judgment and
  verify independently rather than treat the report as a determination.
  This is a deliberate trade of assertiveness for defensibility, consistent
  with the report being described throughout the repo as COPE-aligned
  guidance, not an accusation.
- Coarse tiers mean two references with meaningfully different underlying
  evidence strength can land in the same tier and receive the same weight
  in the aggregate score — the tier system trades some resolution for
  interpretability and gaming-resistance.
