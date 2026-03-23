# Real-Article Test Runs

This directory contains reference lists extracted from real published articles used to validate that the auditor correctly classifies legitimate references as Defensible without over-flagging.

## Validated Articles

| Article | Journal | Year | References | Expected Score | Status |
|---|---|---|---|---|---|
| Hawkins et al. | JOGNN | 2025 | TBD | 75–90 | Tested |
| Dziadkowiec et al. | JOGNN | 2025 | TBD | 75–90 | Tested |
| Cardon & Karr | MCN | 2026 | TBD | 75–90 | Tested |
| Phan, Bethune, & Lathrop | NWH | 2026 | TBD | 75–90 | Tested |

## Adding a Test Article

1. Extract the reference list from the published article.
2. Save as `[first-author-year].md` (e.g., `hawkins-2025.md`).
3. Run the v3 auditor prompt against the reference list.
4. Record the score and any false positives in this index.
5. If false positives are found, document them and evaluate whether the heuristic needs calibration.

## Success Criteria

A real article with a clean reference list should score between **75 and 90**. Scores below 75 suggest over-flagging (false positives). Scores above 90 are unlikely for reference lists of 20+ citations due to the D × 3 base cost in the scoring formula.

Any reference in a real published article that is classified as **High risk** is a false positive and must be investigated. If it indicates a heuristic calibration issue, file a GitHub Issue.
