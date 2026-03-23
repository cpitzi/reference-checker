# Pipeline Architecture Design

This document describes the planned multi-model pipeline decomposition for running the Forensic Reference-Integrity Auditor at editorial scale.

**Status:** Design phase. Not yet implemented. The current system runs as a single Opus prompt.

---

## Why Decompose

The v3 auditor runs as a monolithic Opus prompt. This works for single-manuscript audits, but it has cost and scalability constraints:

- **Token cost:** A full audit of 30–40 references on Opus consumes significant input/output tokens across many tool calls. At editorial scale (dozens of manuscripts per issue cycle), this adds up fast.
- **Processing time:** 5–15 minutes per audit is acceptable for a single manuscript but creates throughput bottlenecks for batch processing.
- **Capability mismatch:** Not every step in the audit requires Opus-level reasoning. DOI resolution is procedural. APA formatting is mechanical. Only forensic interpretation and ambiguous-case judgment genuinely benefit from Opus.

The solution is the same pattern used in any tiered infrastructure: route work to the cheapest resource that can handle it competently, and escalate to more expensive resources only when needed.

---

## Pipeline Stages

### Stage 1: Ingestion and Parsing (Haiku)

**Model:** Haiku
**Input:** Raw reference list (pasted text, extracted from PDF/DOCX, mixed formats)
**Output:** Structured JSON — one object per reference with parsed components (authors, title, journal, year, volume, issue, pages, DOI, URL)

**Why Haiku:** Reference parsing is a structured extraction task. The input format is predictable (citation strings), and the output format is a fixed schema. Haiku handles this reliably and cheaply.

**Failure mode:** Malformed or ambiguous references that Haiku can't parse cleanly get flagged for Stage 3 escalation rather than guessed at.

```json
{
  "ref_id": 1,
  "raw": "Smith, J. A., & Jones, B. C. (2024). Title of article. Journal Name, 45(2), 123-135. https://doi.org/10.1234/example",
  "parsed": {
    "authors": ["Smith, J. A.", "Jones, B. C."],
    "year": "2024",
    "title": "Title of article",
    "journal": "Journal Name",
    "volume": "45",
    "issue": "2",
    "pages": "123-135",
    "doi": "10.1234/example",
    "url": null
  },
  "parse_confidence": "high",
  "missing_fields": []
}
```

### Stage 2: Procedural Verification (Sonnet)

**Model:** Sonnet
**Input:** Structured JSON from Stage 1
**Output:** Verification results per reference — Crossref metadata, PubMed match, retraction status, computed heuristic signals

**Why Sonnet:** Verification is systematic but requires judgment. Sonnet can execute web searches, compare metadata fields, compute string similarity, and flag discrepancies. It doesn't need Opus-level reasoning for straightforward matches/mismatches, but it needs more than Haiku for handling edge cases (name variations, journal title changes, ambiguous DOI resolution).

**Tasks:**
- DOI resolution against Crossref
- PubMed/PMC lookup for biomedical references
- Retraction Watch / Crossref retraction metadata check
- Publisher site verification
- Metadata comparison (title similarity, author list comparison, numeric field matching)
- Heuristic signal computation (homoglyph ratio, Levenshtein distance, author overlap score)

**Output per reference:**
```json
{
  "ref_id": 1,
  "doi_resolves": true,
  "crossref_metadata": { "..." : "..." },
  "pubmed_match": true,
  "retraction_status": "none",
  "heuristic_signals": {
    "title_similarity": 0.98,
    "author_overlap": 1.0,
    "homoglyph_ratio": 0.0,
    "numeric_levenshtein": 0,
    "journal_match": true
  },
  "flags": [],
  "verification_sources": ["crossref", "pubmed"],
  "needs_escalation": false
}
```

**Escalation criteria:** Any reference where:
- Heuristic signals are ambiguous (e.g., title similarity between 85-95%)
- Multiple heuristics trigger simultaneously
- The reference could be a Double-Real composite (DOI resolves but metadata doesn't match)
- Publisher site returns contradictory information vs. Crossref
- The reference matches no authoritative record but looks plausible (shadow-paper candidate)

Escalated references are passed to Stage 3 with full verification context.

### Stage 3: Forensic Interpretation (Opus)

**Model:** Opus
**Input:** Escalated references from Stage 2 (with full verification context), plus summary statistics for the full reference list
**Output:** Final risk classifications, forensic narrative for each escalated reference, pattern analysis across the full list

**Why Opus:** This is the adversarial reasoning stage. Opus evaluates ambiguous cases, interprets the interaction between heuristic signals, assesses whether patterns across multiple references suggest coordinated fabrication, and makes the judgment calls that determine risk tier assignments.

**Tasks:**
- Final risk classification (H/E/M/D) for escalated references
- Cross-reference pattern analysis (do multiple flagged references share characteristics that suggest a common source?)
- Forensic narrative writing (the "Forensic Notes" column in the audit table)
- Executive summary and confidence scoring
- Batch-pattern detection (v4 — analyzing whether flagged references across different manuscripts share fabrication signatures)

**Key principle:** Opus only sees references that *need* its judgment. Clean references that passed Stage 2 without flags don't consume Opus tokens.

### Stage 4: Report Generation (Haiku)

**Model:** Haiku
**Input:** Complete verification results from Stages 2 and 3 — structured data for all references, risk classifications, forensic notes, summary statistics
**Output:** Final HTML report (executive dashboard, forensic audit table, ranked suspicion index, cleaned APA reference list, PRISMA flow diagram, forensic appendix)

**Why Haiku:** Report generation is a templating task. The forensic judgments have already been made; this stage is assembly, formatting, and presentation. Haiku can reliably produce structured HTML from structured input data.

**Tasks:**
- APA 7th edition formatting correction for verified references
- HTML report generation with all six sections
- Confidence gauge calculation from scoring formula
- Risk-tier heatmap rendering
- PRISMA flow diagram assembly from pipeline statistics

---

## Cost Model (Estimated)

Rough token-cost comparison for a 30-reference audit:

| Architecture | Opus Tokens | Sonnet Tokens | Haiku Tokens | Relative Cost |
|---|---|---|---|---|
| **Monolithic Opus (current)** | ~200K | 0 | 0 | 1.0x (baseline) |
| **Decomposed pipeline** | ~30K (escalated refs only) | ~120K | ~50K | ~0.3-0.4x |

Actual savings depend on the escalation rate — what percentage of references require Opus judgment. For clean manuscripts from established researchers, the escalation rate should be low (5-15% of references). For suspicious manuscripts flagged by other tools, the escalation rate will be higher.

**Break-even point:** The pipeline architecture becomes cost-effective when processing more than ~3 manuscripts per month. Below that volume, the monolithic Opus prompt is simpler and adequate.

---

## Implementation Considerations

### Orchestration
The pipeline needs an orchestration layer that:
- Routes references between stages
- Manages escalation decisions
- Aggregates results from all stages into the final report input
- Handles retries and partial failures (e.g., Crossref timeout on one reference shouldn't block the rest)

Options: Direct API orchestration (Python script), LangChain/LangGraph, or a lightweight custom pipeline framework.

### State Management
Each reference accumulates state across stages (parsed fields, verification results, risk classification). This state needs to be passed cleanly between model calls without loss or corruption.

JSON-based structured state is the natural format. Each stage reads the accumulated state, adds its contribution, and passes forward.

### Failure Modes
- **Stage 1 parse failure:** Flag reference, pass raw text to Stage 2 for best-effort verification.
- **Stage 2 verification timeout:** Retry once, then escalate to Stage 3 with "verification incomplete" flag.
- **Stage 3 unavailable:** Fall back to Stage 2 risk classification (more conservative, may over-flag).
- **Stage 4 generation failure:** Retry with simplified template. Report generation should be the most reliable stage.

### Observability
Each stage should emit structured logs:
- References processed / escalated / flagged
- Tool calls made (Crossref, PubMed, web search) with response times
- Token consumption per stage
- Processing time per reference and per stage

This supports both cost monitoring and accuracy analysis — if the pipeline is over-escalating to Opus, the Stage 2 heuristic thresholds need tuning.

---

## Migration Path

1. **Current state:** Monolithic Opus prompt (v3)
2. **Phase 1:** Extract Stage 4 (report generation) into a Haiku post-processing step. Lowest risk, immediate cost savings on the output-generation portion.
3. **Phase 2:** Extract Stage 1 (parsing) into a Haiku pre-processing step. Reduces Opus input tokens.
4. **Phase 3:** Decompose Stage 2 (procedural verification) to Sonnet. This is the largest change and requires the escalation logic.
5. **Phase 4:** Opus receives only escalated references. Full pipeline operational.

Each phase can be tested independently against the adversarial test set and real-article validation set to confirm that decomposition doesn't degrade detection quality.
