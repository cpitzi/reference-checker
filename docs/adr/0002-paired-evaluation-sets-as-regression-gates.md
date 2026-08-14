# ADR-0002: Paired evaluation sets — adversarial detection + real-article specificity — as the regression gate

**Status:** Accepted (2026-06-19; reconstructed 2026-08-13)

## Context

`test-sets/adversarial-30.md` (30 deliberately-crafted bad citations,
present since project scaffold) exercises detection: the auditor should flag
every entry. On its own, that set can't catch a prompt revision that gets
more aggressive and starts over-flagging *clean* references — a change that
looks like an improvement (more flags!) but is actually just noise.

Issue [#3](https://github.com/lentago/reference-checker/issues/3) (opened 2026-03-23, same day as scaffold) tracked
closing that gap; PR [#42](https://github.com/lentago/reference-checker/pull/42) (merged 2026-06-19) delivered it by
adding `test-sets/real-articles/` — full reference lists pulled from two
real, published articles (Ahmadinezhad 2024 and Patriksson 2024), with every
citation individually verified against Crossref before being committed. This
set exercises specificity: it should produce near-zero flags on legitimate
scholarship.

Running the v5 prompt against the new paired sets (PR [#41](https://github.com/lentago/reference-checker/pull/41),
2026-06-19, and the PR #42 real-article run) surfaced a real problem the
detection-only set could never have shown: issue
[#43](https://github.com/lentago/reference-checker/issues/43) found that the scoring formula's `D × 3` base cost
(a per-Defensible-reference deduction) made a *clean* 30-reference article
score only 10, and the clean 26-reference Patriksson article score 22 — both
reading as alarming for articles with no integrity problems. The v6 revision
(PR [#44](https://github.com/lentago/reference-checker/pull/44)) removed the `D × 3` term; the corrected formula
scored the same clean Patriksson article at 96
(`reports/patriksson-2024-v6-2026-06-19.html`), confirmed by the v6 baseline
run (PR [#46](https://github.com/lentago/reference-checker/pull/46), 2026-06-20).

CLAUDE.md states the operating principle this issue established directly: "a
prompt revision that improves one [set] without regressing the other is a
real win; a revision that flags more on both is just noisier."

## Decision

Every prompt revision is evaluated against both test sets together before
being treated as a production baseline: `adversarial-30.md` for detection
(sensitivity) and `test-sets/real-articles/` for false-positive rate
(specificity). Baselines — HTML reports plus a metrics/verdict writeup — are
committed to `reports/` per version (v4: PR #36/#38; v5: PR #41; v6: PR #46),
so every shipped version has a reproducible, comparable record. A revision
that raises flags on the real-article corpus without a matching detection
gain on the adversarial set is treated as a regression, not an improvement,
regardless of how it scores on the adversarial set alone.

## Alternatives

**Recorded at the time:**
- **Detection-only evaluation** (the status quo before PR #42): keep
  `adversarial-30.md` as the sole gate. Cheaper to run, but issue #43's
  finding — a structural scoring bug that punished clean articles — was
  invisible to it by construction; a detection-only set has no clean
  reference lists to over-penalize.

**Retrospective — not considered at the time:**
- **A single blended metric** (e.g., one combined score averaging detection
  and specificity results) instead of two separately-reported test-set runs.
  *Lateral, arguably worse*: averaging would have partially masked exactly
  the failure issue #43 caught — a formula that tanks clean-article scores
  could still average out to a passable blended number if adversarial
  detection stayed strong. Keeping the two signals separate and both
  visible in `reports/README.md` is what let the Patriksson 22→96 correction
  register as an unambiguous, specific fix rather than a rounding change.
- **Statistical sampling from a larger corpus of real articles** rather than
  two hand-verified ones. *Better for statistical power, worse for this
  project's constraints*: each real-article entry requires individually
  verifying every citation against Crossref to avoid the auditor's own
  anti-fabrication protocol backfiring on its test data (documented in the
  #42 PR body); two carefully-verified articles are tractable for a
  single-operator lab project in a way a larger sampled corpus isn't yet.

## Consequences

- Shipping a new prompt version now implies two evaluation runs, not one —
  more work per revision, but it already caught one real, non-obvious defect
  (the `D × 3` scoring bug) that pure detection testing would have missed
  entirely.
- `reports/` accumulates paired baselines per version (adversarial + both
  real articles), which is the reproducibility record referenced in
  [ADR-0001](0001-prompt-engineered-product-versioned-prompts.md) — there's
  no other regression history for a prompt-only product.
- The real-article corpus is expensive to grow (each addition needs
  individual Crossref verification), so it stays small; specificity
  measurement has less statistical power than detection measurement on the
  30-entry adversarial set.
