# 12 — Evaluation rubric

The rubric totals 100 points. There are no bonus points.

| Category | Points | Evidence |
|---|---:|---|
| Functional and data correctness | 40 | Five Must requirements; canonical 18/2 result; ordered tasks; clear errors and failure stops |
| Data model and ETL practice | 25 | SQLite keys/grain, transactional load, quarantine reasons, checksum/date identity, replay safety |
| Tests and CI | 15 | 20 distinct cases, one Python 3.12.x CI job, named package coverage ≥80% line / ≥70% branch |
| Local reproducibility | 10 | Exact CLI works offline after dependency setup; local state and deterministic report are inspectable |
| Documentation and Git | 5 | Concise README, traceable changes, clean pull-request workflow, no generated state or secrets committed |
| Demo and viva | 5 | Five-minute demo and clear explanation of design, tests, retry, and idempotency |
| **Total** | **100** | |

## Must pass gates

All gates are achievable within the 30-hour plan:

1. The exact demo command succeeds on the fictional 20-row fixture and reports 18 accepted / 2 quarantined.
2. Exactly three tasks are recorded in order; failure prevents downstream work; the injected transient read error is retried once.
3. The same file checksum and business date do not create duplicate per-run rows; the report matches SQLite.
4. The 20 specified tests pass and the named package meets 80% line / 70% branch coverage.
5. The student can explain the implementation and its failure behavior.

[Back to README](../README.md)
