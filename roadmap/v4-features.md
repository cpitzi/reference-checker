# v4 Planned Features

This document describes the heuristics and capabilities planned for v4 of the Forensic Reference-Integrity Auditor. Each feature includes a description, implementation approach, priority assessment, and dependencies.

**Status:** Planning. No v4 features have been implemented or tested yet.

---

## New Heuristics

### Sneaked-Reference Detection

**Priority:** High (recommended first implementation)

**What it catches:** References that appear in the reference list but are never cited in the manuscript body. This is a reference-list padding technique — adding citations to inflate apparent scholarly engagement or to boost citation counts for specific papers.

**Implementation approach:**
- Requires access to the full manuscript body, not just the reference list
- Parse in-text citations from the manuscript (APA parenthetical and narrative citation patterns)
- Cross-reference parsed in-text citations against the reference list
- Flag any reference list entries that have no corresponding in-text citation
- Handle edge cases: footnote citations, table/figure source notes, supplementary material references

**Why high priority:** This is the simplest v4 heuristic to implement and test. It requires no new external data sources (just the manuscript text), the logic is straightforward string matching, and it catches a common and well-documented form of citation manipulation. It's also immediately testable against any full manuscript.

**Dependencies:** The auditor must receive the full manuscript, not just the reference list. This changes the input requirements.

---

### Temporal Impossibility Checks

**Priority:** High

**What it catches:** Citations with dates that are logically impossible:
- Reference year predates the journal's founding year
- Reference year postdates the manuscript submission date
- Volume/issue numbers that don't exist for the stated year
- References to "forthcoming" papers that were never published

**Implementation approach:**
- Maintain or query a lookup of journal founding dates (Crossref, ISSN portal, Ulrichsweb)
- Compare reference year against journal founding year
- Compare reference year against manuscript submission/acceptance date (if available)
- Cross-check volume/issue against Crossref's year-to-volume mapping for the journal
- For "in press" or "forthcoming" citations, verify that the paper was eventually published

**Why high priority:** Temporal impossibility is an unambiguous signal. A paper published in a journal that didn't exist yet is definitively fabricated, no judgment call required. Low false-positive rate, high signal value.

**Dependencies:** Journal founding date data. Crossref provides some of this, but coverage is incomplete for smaller or newer journals.

---

### Predatory Journal Flagging

**Priority:** Medium

**What it catches:** References to journals identified as predatory or questionable by established assessment methodologies (primarily Cabells Predatory Reports).

**Implementation approach:**
- Cross-reference journal titles against known predatory journal lists
- Check for Cabells classification (if accessible)
- Apply heuristic indicators: journal not indexed in PubMed/Scopus/Web of Science, no COPE membership, suspicious publisher practices (rapid peer review claims, aggressive solicitation patterns)
- Flag but do not auto-classify as High risk — some legitimate research is published in questionable venues

**Complexity notes:**
- "Predatory" is a contested term with no universal definition
- Cabells Predatory Reports is behind a paywall and may not be programmatically accessible
- Beall's List (discontinued, archived) is outdated and controversial
- The heuristic should flag for editorial awareness, not make a definitive judgment
- DOAJ (Directory of Open Access Journals) inclusion is a positive signal but not definitive

**Why medium priority:** Valuable signal but complex to implement well. The risk of false positives (legitimate open-access journals flagged as predatory) requires careful calibration. Editorial sensitivity around this topic is high.

**Dependencies:** Access to journal classification data. May require manual maintenance of a supplementary journal list.

---

### Batch-Pattern Detection

**Priority:** Medium

**What it catches:** Statistical patterns across multiple manuscripts that suggest coordinated fabrication — the same fabricated references appearing in different submissions, unusual citation clustering, or shared fabrication signatures.

**Implementation approach:**
- Maintain a database of previously audited references and their verification results
- When auditing a new manuscript, compare its reference list against the historical database
- Flag references that appear in multiple submissions with identical anomalies
- Detect citation rings (manuscripts that exclusively cite each other)
- Identify shared fabrication patterns (same homoglyph substitutions, same digit-swap patterns, same shadow-paper templates)

**Why medium priority:** High value for publishers processing many manuscripts, but requires persistent state across audit runs — a significant architectural change from the current stateless prompt-based approach. This is more of a platform feature than a prompt feature.

**Dependencies:** Persistent storage, historical audit data, statistical analysis capabilities beyond what a single prompt session can provide.

---

### Crossref Retraction API Integration

**Priority:** Medium

**What it catches:** Papers that have been retracted or have expressions of concern, using Crossref's structured retraction metadata rather than relying on web search of Retraction Watch.

**Implementation approach:**
- Query Crossref API for retraction metadata on each DOI
- Check the `update-to` field for retraction notices
- Check the `is-referenced-by` field for expressions of concern
- Cross-reference with Retraction Watch for cases not yet reflected in Crossref metadata

**Why medium priority:** The current v3 auditor already checks for retractions via web search. Direct API integration would be more reliable and faster, but it's an optimization of existing capability rather than new detection capability.

**Dependencies:** Crossref API access (free for polite use with mailto header). Pipeline architecture (Stage 2) would be the natural integration point.

---

### COPE Flowchart Alignment

**Priority:** Low (but high value for productization)

**What it catches:** N/A — this is an output enhancement, not a detection heuristic.

**What it does:** Structures the auditor's recommendations according to COPE (Committee on Publication Ethics) investigation flowcharts. When the auditor identifies probable fabrication, the output would include the specific COPE flowchart that applies and the recommended investigation steps.

**Implementation approach:**
- Map risk classifications to COPE flowchart entry points
- For High-risk references: link to COPE's "Suspected fabricated data" flowchart
- For retracted references: link to COPE's "Systematic manipulation of the publication process" flowchart
- Include recommended next steps aligned with COPE procedures (contact author, contact institution, etc.)

**Why low priority:** This is a presentation/documentation enhancement. It doesn't improve detection — it improves the editorial workflow after detection. High value for productization because it demonstrates alignment with industry governance standards.

**Dependencies:** Thorough understanding of COPE flowchart logic and when each applies. COPE guidelines are publicly available.

---

## Architecture Features

### API-Based Orchestration

**Priority:** High (prerequisite for other features)

Move from prompt-based execution to API-based pipeline orchestration as described in `docs/architecture.md`. This enables:
- Multi-model pipeline (Haiku → Sonnet → Opus → Haiku)
- Cost optimization through model tiering
- Batch processing of multiple manuscripts
- Persistent state for batch-pattern detection

### Token Budget Management

**Priority:** Medium

Implement token consumption tracking and optimization:
- Per-stage token metering
- Adaptive escalation thresholds (reduce Opus usage when budget is constrained)
- Cost-per-audit reporting
- Budget alerts for editorial teams

### Report Template System

**Priority:** Low

Make the HTML report output customizable:
- Journal-specific branding and formatting preferences
- Configurable report sections (some editors may not need all six sections)
- Export formats beyond HTML (PDF, structured JSON for integration with editorial management systems)
- JATS XML output for direct integration with journal production workflows

---

## Prioritized Implementation Order

1. **Sneaked-reference detection** — Highest signal-to-effort ratio. Testable immediately.
2. **Temporal impossibility checks** — Unambiguous signals, low false-positive rate.
3. **API-based orchestration** — Prerequisite for scaling and for batch-pattern detection.
4. **Crossref retraction API integration** — Optimization of existing capability.
5. **Predatory journal flagging** — Valuable but requires careful calibration.
6. **Batch-pattern detection** — High value but requires persistent state infrastructure.
7. **COPE flowchart alignment** — Presentation enhancement for productization.
8. **Token budget management** — Operational optimization.
9. **Report template system** — Customization for publisher adoption.
