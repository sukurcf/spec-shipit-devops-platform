# 13 — Interview preparation

## One-minute explanation

Explain that DataFlow is a small local batch runner with three fixed tasks: validate a CSV, persist accepted and rejected rows in SQLite, and write a deterministic JSON report. Describe the 20-row fixture and its 18/2 result, the checksum-plus-business-date run ID, one injected transient read retry, and why downstream work stops on failure. Close with how tests prove ordering, status, replay safety, and report/database agreement.

## Questions and key points

1. **Why keep the runner fixed at three tasks?** The goal is to learn sequencing and failure handling without building a general scheduler.
2. **What makes a run idempotent?** The logical key combines source SHA-256 and business date; composite SQLite row keys prevent same-run duplicates.
3. **Why hash the source bytes?** Identical file content gets the same identity regardless of a later read; changing any byte changes the checksum.
4. **Which data errors are quarantined?** Invalid rows such as missing IDs, malformed dates, and invalid amounts; they retain a reason.
5. **Which errors stop the run?** Unreadable input, invalid header/date, and task failures; downstream tasks are skipped.
6. **Why retry only a transient read error?** One retry can recover a temporary read interruption; repeating validation or storage failures would hide a real defect.
7. **How is money represented?** Parse decimal text and store accepted values as integer cents to avoid binary floating-point rounding.
8. **How is the SQLite load safe?** Accepted and quarantined rows are written transactionally so a failed load does not leave a partial batch.
9. **How is the report deterministic?** Stable run ID, sorted JSON keys, fixed task ordering, and no timestamps or absolute paths.
10. **How do tests prove the summary is correct?** Compare report counts against per-run accepted and quarantined rows in SQLite, then replay the same input and verify no growth.

[Back to README](../README.md)
