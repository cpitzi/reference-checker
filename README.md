# Forensic Reference-Integrity Auditor

A prompt-engineered deep-scan verification system for academic reference lists. Built for managing editors in nursing and health sciences publishing who need to catch fabricated, manipulated, and suspicious citations before they reach print.

## The Problem

Academic reference lists are a trust surface. Paper mills, AI-generated citations, and increasingly sophisticated metadata manipulation mean that a reference can *look* perfectly formatted while being completely fabricated — or worse, a composite of real elements assembled to resist casual verification.

Existing tools address slices of this problem:

| Tool | What It Does | What It Misses |
|---|---|---|
| **Edifix** | Formatting correction, DOI lookup | No adversarial verification |
| **Scite.ai** | Citation context analysis | Doesn't detect fabricated metadata |
| **iThenticate** | Text similarity / plagiarism | Ignores reference list integrity |
| **Papermill Alarm** | Paper mill pattern detection | Narrow heuristic scope |
| **RefChecker** | Basic DOI/metadata validation | No forensic depth |

None of them perform adversarial forensic verification across multiple heuristic dimensions simultaneously. That's what this tool does.

## How It Works

The auditor runs as a structured prompt on Anthropic's Claude (Opus), using live web search to verify every citation against authoritative sources:

- **Crossref** — DOI resolution, metadata matching, retraction status
- **PubMed / PMC** — Biomedical citation verification
- **Retraction Watch** — Known retraction and expression-of-concern database
- **Publisher sites** — Direct verification against journal archives

### Forensic Heuristics (v3)

Each reference is evaluated against seven forensic heuristics designed to catch progressively more sophisticated fabrication:

| # | Heuristic | What It Catches |
|---|---|---|
| 1 | **DOI Resolution** | Dead DOIs, DOIs pointing to wrong papers, fabricated DOI patterns |
| 2 | **Homoglyph Detection** | Cyrillic or other Unicode substitutions in titles, author names, or journal names designed to defeat string matching |
| 3 | **Digit-Swap Analysis** | Transposed volume/issue/page numbers that make a real citation unfindable |
| 4 | **Author-Shifting** | Subtly rearranged, added, or removed authors compared to the actual publication record |
| 5 | **Double-Real Trap** | Real DOI + real-sounding metadata from a *different* paper, creating a composite that passes surface checks |
| 6 | **Journal Mutation** | Slightly altered journal titles (word substitution, abbreviation manipulation) that point to nonexistent or different journals |
| 7 | **Shadow-Paper Signatures** | Citations with plausible metadata that match no known publication — fully fabricated but constructed to look legitimate |

### Risk Classification

Every reference receives one of four risk tiers:

| Tier | Label | Meaning |
|---|---|---|
| **H** | High | Strong evidence of fabrication or manipulation. Recommend rejection or author query. |
| **E** | Elevated | Multiple anomalies detected. Requires manual verification before acceptance. |
| **M** | Moderate | Minor anomalies or incomplete verification. Flag for editorial awareness. |
| **D** | Defensible | Verified or consistent with known publication records. No action required. |

### Scoring Formula

```
Reference List Score = 100 − (H × 12) − (E × 5) − (M × 2) − (D × 3)
```

The weights punish fabrication heavily while avoiding over-penalization of grey literature (government reports, organizational white papers, URLs) that legitimately lacks DOIs.

## Output

The auditor produces a self-contained HTML report with six sections, designed for editorial decision-making:

1. **Executive Dashboard** — Confidence gauge (0–100), risk-tier heatmap, summary stat cards. A managing editor can glance at this and know whether to worry.
2. **Forensic Audit Table** — Per-reference findings with heuristic flags, verification sources consulted, and risk tier assignments.
3. **Ranked Suspicion Index** — References ordered by risk severity. Highest-risk citations surface first.
4. **Cleaned APA Reference List** — Corrected formatting for all verified references (APA 7th edition).
5. **PRISMA-Style Flow Diagram** — Visual representation of how references moved through the verification pipeline (verified, flagged, unresolvable, grey literature).
6. **Forensic Appendix** — Methodology documentation, heuristic definitions, and scoring explanation. Supports editorial audit trails and COPE-aligned documentation.

## Usage

### Requirements

- Anthropic Claude (Opus recommended for forensic interpretation quality)
- Web search enabled (the auditor performs live verification against external sources)

### Running an Audit

1. Provide the prompt (see `prompts/v3-auditor.md`) to Claude with web search enabled.
2. Paste or upload the reference list to be audited.
3. The auditor will systematically verify each reference and produce the HTML report.

> **Note:** A single audit of 25–40 references typically requires 5–15 minutes of processing time and significant tool-call volume. This is by design — thorough forensic verification is not a quick-check operation.

### Input Formats

The auditor accepts reference lists in:

- Raw text (pasted APA-formatted references)
- Extracted from manuscript PDFs or Word documents
- Mixed formats (the auditor will normalize during processing)

## Testing

The system has been validated against:

### Adversarial Test Set
A deliberately constructed 30-reference list containing layered traps:
- Homoglyph substitutions (Cyrillic characters in journal titles)
- Author-shifted citations (real papers with manipulated author lists)
- Shadow papers (fully fabricated but plausible-sounding)
- Double-Real composites (real DOI + metadata from a different paper)
- Pop-culture junk citations (including a fabricated Obi-Wan Kenobi publication)
- Clean references seeded throughout to test false-positive rates

### Real Published Articles
Multiple real articles from JOGNN, MCN, and related nursing journals verified to confirm that the auditor correctly classifies legitimate references as Defensible without over-flagging.

## Roadmap

### v4 Heuristics (Planned)
- **Batch-pattern detection** — Statistical analysis across multiple submissions to identify coordinated fabrication campaigns
- **Crossref Retraction API integration** — Direct programmatic retraction checking
- **Predatory journal flagging** — Cabells-style methodology for identifying predatory or questionable venues
- **Temporal impossibility checks** — Citations with dates that predate the journal's existence or postdate the submission
- **Sneaked-reference detection** — References that appear in the list but are never cited in the manuscript body
- **COPE flowchart alignment** — Structured recommendation output aligned with Committee on Publication Ethics investigation procedures

### Architecture (Planned)
Pipeline decomposition across model tiers for cost optimization at editorial scale:

| Stage | Model | Role |
|---|---|---|
| Forensic interpretation | Opus | Judgment calls, ambiguous cases, adversarial reasoning |
| Procedural verification | Sonnet | DOI resolution, metadata matching, systematic checks |
| Formatting and output | Haiku | APA correction, HTML report generation, structured output |

## Project Context

This project originated from a real editorial workflow need. My sister Jennifer is a managing editor at [AWHONN](https://www.awhonn.org/) (Association of Women's Health, Obstetric and Neonatal Nurses), which publishes *JOGNN*, *MCN*, and *Nursing for Women's Health*. These journals face the same reference-integrity threats as all academic publishing, amplified by the rapid growth of AI-generated content and paper mill sophistication.

The tool is designed to fit into a managing editor's actual workflow: receive a manuscript, run the reference list through the auditor, get a report that supports an editorial decision. Not a research tool — an editorial operations tool.

## Development Methodology

This project uses **imperative-to-declarative promotion** as its core development methodology:

1. **Exploratory run** — Execute the prompt, observe what Claude produces, optimize for good raw output.
2. **Identify what works** — Name the specific behaviors, heuristics, and output patterns that succeeded.
3. **Codify into spec** — Write the successful behavior into the prompt as declarative instructions that any Claude instance can reproduce cold.

This is the same pattern as writing configuration management (Puppet, Ansible) from a hand-tuned known-good state: get the system working by hand, then capture that state as code.

Nothing gets added to the spec until it's been tested. The prompt is the artifact.

## Repository Structure

```
├── README.md
├── prompts/
│   └── v3-auditor.md          # Current production prompt
├── test-sets/
│   ├── adversarial-30.md      # Adversarial reference list with layered traps
│   └── real-articles/         # Real article reference lists used for validation
├── reports/                   # Sample output reports
├── docs/
│   ├── heuristics.md          # Detailed heuristic documentation
│   ├── competitive-landscape.md
│   └── architecture.md        # Pipeline decomposition design
└── roadmap/
    └── v4-features.md         # Planned enhancements
```

## License

MIT License — see [LICENSE](LICENSE).

## Credits

Built by Chris Pitzi — infrastructure professional turned prompt engineer. 30 years of production operations applied to making AI do useful, verifiable, adversarial work. Developed with [Claude](https://claude.ai) (Anthropic).
