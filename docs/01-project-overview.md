# 01 — Project overview

## Problem and outcome

Data engineers need to turn small file drops into checked, repeatable datasets. DataFlow is a learning project in which one fixed Python runner validates a fictional orders CSV, stores accepted and rejected rows in SQLite, and writes a JSON summary.

By the end of five days (30 hours), the demo processes 20 source rows: 18 accepted, 2 quarantined. A successful run records three ordered task statuses. Repeating the same file checksum and business date adds no target rows.

## Scope

**In scope:** Python 3.12.x; standard-library CSV, SQLite, JSON, date, decimal, and checksum handling; one local CLI; one SQLite file; pytest; deterministic fixture and outputs.

**Out of scope:** more than three pipeline tasks, a general scheduler, remote or distributed execution, network calls, listening ports, user interfaces, and external data sources.

## Success criteria

- The exact command and output contract in [document 08](08-platform-specification.md) work from a prepared local checkout.
- A valid-header input is classified row by row; bad rows are quarantined with a reason.
- A file-level or task failure prevents downstream work and leaves inspectable status.
- The same input checksum and business date produce the same logical run ID and no duplicate rows.
- The tests cover the required cases and meet the named package coverage target.

## Skill-to-job mapping

| DataFlow work | Entry-level data-engineering skill |
|---|---|
| Read a delimited file and enforce a schema | File ingestion and data contracts |
| Validate dates, identifiers, and money | Data quality and validation |
| Persist rows and run state in SQLite | Relational modeling and transactions |
| Order tasks and stop on failure | Batch orchestration fundamentals |
| Reconcile a JSON summary to stored rows | Operational reporting and verification |
| Test errors, retries, and replay | Reliable pipeline testing |

The specification defines acceptance criteria only; students write implementation code in their own repository.

[Back to README](../README.md)
