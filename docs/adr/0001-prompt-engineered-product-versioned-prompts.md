# ADR-0001: The product is a versioned prompt program, not a scripted pipeline

**Status:** Accepted (2026-03-23; reconstructed 2026-08-13)

## Context

The auditor's core logic lives entirely in a single Opus prompt that uses live
web search against Crossref, PubMed, Retraction Watch, publisher sites, and
the NLM Catalog to verify citations. There is no compile step, test runner,
or package manager — as [CLAUDE.md](../../CLAUDE.md) puts it, "the 'build
system' is 'paste the prompt into Claude with a reference list.'"

At project scaffold (PR [#1](../../pull/1), 2026-03-23) the live prompt was
moved into `prompts/v3-auditor.md`, establishing the convention that ships to
this day: each revision lands as the next-integer file
(`prompts/v<N>-auditor.md`), never an in-place edit, so old versions stay
readable for diffing and for reproducing past audit runs. Four versions have
shipped under this convention since: v3 (scaffold), v4 (2026-05-21), v5
(PR [#39](../../pull/39), 2026-06-19), and v6 (PR [#44](../../pull/44),
2026-06-19).

The same scaffold commit added [`docs/architecture.md`](../architecture.md),
a design document for decomposing the monolithic Opus prompt into a
four-stage Haiku/Sonnet/Opus/Haiku pipeline for editorial-scale throughput,
and issue [#11](../../issues/11) to track implementing it. Both the design
doc and the issue are still open as of this writing — the pipeline is
deliberately designed but not built. `docs/architecture.md` estimates the
decomposed pipeline only becomes cost-effective above roughly 3 manuscripts/
month, and the live-web-search verification surface the whole heuristic set
depends on (see the "Live web search is the verification surface" gotcha in
CLAUDE.md) is easiest to reason about as a single adversarial-reasoning
session rather than state handed between model tiers.

## Decision

Keep the auditor as a single versioned Opus prompt. Ship every revision as a
new `prompts/v<next>-auditor.md` file rather than mutating the current one.
Treat the multi-model pipeline decomposition as a scoped-but-deferred design
(tracked in `docs/architecture.md` and issue #11), not a roadmap commitment.

## Alternatives

**Recorded at the time:**
- **Decompose into a Haiku (parse) → Sonnet (verify) → Opus (interpret) →
  Haiku (report) pipeline**, per `docs/architecture.md`. Estimated ~0.3–0.4x
  the token cost of the monolithic approach, but requires an orchestration
  layer, inter-stage JSON schema contracts, and escalation-criteria tuning
  that didn't exist yet. Explicitly deferred pending real throughput demand
  ("editorial scale," estimated break-even above ~3 manuscripts/month) that
  hasn't materialized against a single-Opus-prompt lab project.

**Retrospective — not considered at the time:**
- **A scripted tool calling the Crossref/PubMed/Retraction Watch APIs
  directly**, with no LLM in the verification loop at all. *Worse fit for
  this project's actual purpose*: it would likely be more deterministic on
  the mechanical checks (DOI resolution, retraction lookups), but it drops
  the exact thing the repo exists to demonstrate — prompt-engineering-as-
  product — and it can't do the adversarial pattern-interpretation the
  forensic heuristics (homoglyph substitution, author-shifting, shadow-paper
  signatures) actually depend on. Live web search *is* the verification
  surface by design, not an implementation detail to be optimized away.

## Consequences

- Every prompt revision is fully readable and diffable against every prior
  version; reproducing a past audit run means pointing at the matching
  `prompts/v<N>-auditor.md` file, not reconstructing pipeline state.
- There is no automated regression suite in the software-engineering sense —
  quality control runs through the paired evaluation sets described in
  [ADR-0002](0002-paired-evaluation-sets-as-regression-gates.md), executed by
  hand (or by an agent) against each new version.
- The pipeline decomposition stays a live option, not a dead idea: if
  editorial-scale throughput ever becomes a real requirement, `docs/
  architecture.md` and issue #11 are the starting point rather than a
  from-scratch design effort.
- Cost and latency scale linearly with reference-list size on every run,
  since there's no cheap-tier pre-filtering of obviously-clean references.
