# Forensic Heuristics — Detailed Documentation

This document describes each forensic heuristic used by the Reference-Integrity Auditor, including what it detects, how it works, known limitations, and examples from testing.

All heuristics are applied per-reference during the verification pass. A single reference can trigger multiple heuristics simultaneously — and frequently does in adversarial test sets.

---

## 1. DOI Resolution

**What it catches:** Dead DOIs, DOIs that resolve to a different paper than described, fabricated DOI patterns that follow the `10.xxxx/` convention but point nowhere.

**How it works:** The auditor resolves each DOI against Crossref. If the DOI is present, three outcomes are possible:
- **Resolves correctly** — The Crossref metadata matches the supplied citation. No flag.
- **Resolves to wrong paper** — The DOI is real but belongs to a different publication. This is a strong fabrication signal (see Heuristic 5, Double-Real Trap).
- **Does not resolve** — The DOI returns no Crossref record. Could indicate a fabricated DOI, a typo, or a DOI from a registry Crossref doesn't index (DataCite, etc.).

**Limitations:**
- Some legitimate DOIs are registered through DataCite (datasets, code) rather than Crossref.
- Very recently published papers may have DOIs that haven't propagated to Crossref yet.
- Book chapter DOIs sometimes resolve to the parent volume rather than the specific chapter.

**Grey-literature note:** Government reports, organizational guidelines, and web resources typically lack DOIs. The absence of a DOI is not itself a flag — only the presence of a DOI that fails to resolve triggers this heuristic.

---

## 2. Homoglyph Detection

**What it catches:** Unicode character substitutions designed to defeat automated string matching. The most common pattern is Cyrillic-to-Latin substitution (Cyrillic `а` for Latin `a`, Cyrillic `о` for Latin `o`, etc.), but the heuristic also covers other confusable character sets.

**How it works:** The auditor computes a suspicious-character ratio for each reference's title, author names, and journal name. Characters are checked against Unicode confusables tables. A reference with a ratio above threshold is flagged.

**Why it matters:** Homoglyph substitution is a paper mill technique. A fabricated reference with a Cyrillic `е` in the journal title will pass a visual inspection but fail an exact-match lookup against Crossref or PubMed, making the fabrication harder to catch by automated tools that rely on string matching — while still looking correct to a human reviewer.

**Limitations:**
- Author names from non-Latin-script languages may contain legitimate Unicode characters that trigger false positives if the threshold is set too aggressively.
- Some legitimate journals have titles that include non-ASCII characters by design.
- The heuristic catches the substitution but cannot determine intent — a genuine encoding error looks identical to deliberate manipulation.

**Test evidence:** The adversarial test set includes references with Cyrillic `а` and `о` substituted into journal titles of real nursing journals. The auditor successfully flagged these while passing clean references with identical journal names in standard Latin characters.

---

## 3. Digit-Swap Analysis

**What it catches:** Transposed or altered volume numbers, issue numbers, page ranges, or article numbers that make an otherwise real citation unfindable through standard lookup.

**How it works:** When Crossref or PubMed metadata is available for a reference, the auditor computes the Levenshtein distance between the supplied numeric tokens (volume, issue, pages) and the authoritative values. A distance of 1 (single transposition or substitution) is flagged as a probable digit-swap; a distance >1 is flagged as a more significant discrepancy.

**Why it matters:** Digit-swapping is a low-effort manipulation technique that creates plausible deniability ("it was a typo"). A reference to volume 48 instead of volume 84 looks like carelessness, but it makes the citation unverifiable and may indicate that the author never consulted the original source.

**Limitations:**
- Genuine typos are indistinguishable from deliberate manipulation at the single-reference level. The heuristic flags both; editorial judgment determines which is more likely.
- Online-first articles with changing page numbers may show legitimate discrepancies between the supplied citation and the Crossref record.
- Some journals have both a volume/issue system and an article-number system, and authors sometimes mix them.

---

## 4. Author-Shifting

**What it catches:** Author lists that have been subtly manipulated compared to the actual publication record — names added, removed, reordered, or replaced.

**How it works:** The auditor compares the supplied author list against the authoritative Crossref or PubMed record. Discrepancies are categorized:
- **Author added** — A name appears in the supplied reference that is not in the authoritative record.
- **Author removed** — A name from the authoritative record is missing from the supplied reference.
- **Author reordered** — The same names appear but in a different sequence.
- **Author substituted** — A name in the supplied reference replaces a different name from the authoritative record (may indicate name confusion or deliberate substitution).

**Why it matters:** Author-shifting serves multiple purposes in academic fraud:
- Adding a fabricated co-author to make the reference appear connected to a different research group.
- Removing authors to obscure the true provenance of a paper.
- Reordering to change the apparent first/last author (significant in academic credit conventions).

**Limitations:**
- Author name variations (initials vs. full names, transliterated names, hyphenation differences, maiden vs. married names) create legitimate discrepancies that are not manipulations.
- Large collaborative papers (50+ authors) may have author lists that differ between Crossref and PubMed due to metadata processing differences.
- Pre-print to final publication author list changes are legitimate.

---

## 5. Double-Real Trap

**What it catches:** A citation constructed by pairing a real DOI from one paper with metadata (title, authors, journal) from a different real paper, creating a composite that passes surface-level checks against either source individually.

**How it works:** When a DOI resolves successfully, the auditor cross-checks the resolved metadata against the supplied metadata. If the DOI resolves but the metadata matches a *different* paper, the reference is flagged as a probable Double-Real composite.

**Why it matters:** This is one of the more sophisticated fabrication techniques. Both the DOI and the metadata are independently real, so tools that check only DOI resolution (does the DOI exist?) or only metadata plausibility (does this title sound real?) will miss the mismatch. You have to check both and compare them to each other.

**Limitations:**
- Requires both Crossref metadata and sufficient supplied metadata to perform the cross-check. References with minimal metadata (no title, only DOI) cannot be fully evaluated.
- Erratum and correction notices sometimes have DOIs that resolve to related but different content.

**Test evidence:** The adversarial test set includes multiple Double-Real composites pairing DOIs from high-profile nursing research papers with metadata from different papers in the same journal. The auditor detected the mismatch by comparing Crossref-returned titles against supplied titles.

---

## 6. Journal Mutation

**What it catches:** Journal titles that have been subtly altered — word substitutions, abbreviation changes, added/removed qualifiers — that point to nonexistent journals or to different journals than intended.

**How it works:** The auditor compares the supplied journal title against known journal title registries (Crossref, PubMed/NLM catalog, ISSN portal). String similarity is computed; titles below a 95% match threshold are flagged for review.

**Common mutation patterns:**
- Word substitution: "Journal of Obstetric, Gynecologic & Neonatal Nursing" → "Journal of Obstetric, Gynecologic & Perinatal Nursing"
- Abbreviation manipulation: "JOGNN" → "J Obstet Gynecol Neonat Nurs" (plausible but non-standard abbreviation)
- Qualifier addition: "International Journal of Nursing Studies" → "International Journal of Advanced Nursing Studies"
- Scope change: "Nursing for Women's Health" → "Nursing for Maternal Health"

**Why it matters:** Mutated journal titles create citations that look legitimate on casual inspection but cannot be verified against any real publication record. This is a common paper mill technique for manufacturing citations to fabricated papers.

**Limitations:**
- Journals that have changed names over time (common in academic publishing) may trigger false positives when older citations use the historical title.
- Some journals have very similar titles that are distinct publications (e.g., "Nursing Research" vs. "Research in Nursing & Health").

---

## 7. Shadow-Paper Signatures

**What it catches:** Citations with fully plausible metadata — reasonable title, real-sounding authors, appropriate journal, correct formatting — that match no known publication in any authoritative database.

**How it works:** After all other heuristics have been applied, the auditor performs an exhaustive search for the reference across Crossref, PubMed, the publisher's site, Google Scholar, and library catalogs. If no record is found anywhere, the reference is classified as a probable shadow paper.

**Why it matters:** Shadow papers are fully fabricated citations constructed to resist casual verification. Unlike crude fabrications (obviously fake journal titles, impossible dates), shadow papers use:
- Author names from real researchers in the field
- Journal titles that exist
- Volume/issue numbers that are plausible for the publication year
- Titles that sound like real research in the subject area

The only way to catch them is exhaustive verification — checking that the specific paper actually exists, not just that the components are plausible.

**Limitations:**
- Very new publications may not yet be indexed in all databases.
- Conference proceedings, dissertations, and institutional reports may be poorly indexed and appear to be shadow papers when they are legitimate.
- Publications in non-English-language journals may be harder to verify and may be misclassified.
- The classification depends on the exhaustiveness of the search. A shadow paper can only be identified with confidence after multiple authoritative sources return no results.

**Test evidence:** The adversarial test set includes shadow papers constructed using real nursing researcher names, real JOGNN volume/issue numbers, and titles modeled on actual published research in the field. The auditor correctly identified them as unverifiable after exhaustive search.

---

## Heuristic Interaction

These heuristics are not independent — they interact and compound:

- A **Double-Real Trap** will pass DOI Resolution (the DOI is real) but fail on Author-Shifting or title mismatch when cross-checked.
- A **Shadow Paper** with a fabricated DOI will trigger both DOI Resolution failure and Shadow-Paper classification.
- A **Homoglyph** in a journal title may also trigger Journal Mutation if the substitution creates a string that doesn't match the real journal name.
- **Digit-Swap** combined with **Author-Shifting** on the same reference is a strong coordinated-fabrication signal.

The risk classification system accounts for these interactions. A reference triggering multiple heuristics receives a higher risk tier than one triggering a single heuristic in isolation.

---

## Version History

| Version | Changes |
|---|---|
| **v1** | DOI resolution only |
| **v2** | Added homoglyph detection, digit-swap analysis, shadow-paper signatures |
| **v3** | Added author-shifting, Double-Real trap detection, journal mutation. Refined scoring formula. Added grey-literature handling. |
| **v4 (planned)** | Batch-pattern detection, temporal impossibility checks, sneaked-reference detection, predatory journal flagging |
