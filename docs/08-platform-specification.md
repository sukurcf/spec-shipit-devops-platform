# 08 — DataFlow task-runner specification

This document defines the only required business interface. The runner has no network interface and is not a general-purpose scheduler.

## CLI contract

Run from the student's implementation repository:

```text
uv run dataflow run fixtures/orders.csv --date 2026-10-13 --db .local/dataflow.db --report .local/report.json
```

| Argument | Contract |
|---|---|
| `run` | Execute exactly three tasks in the fixed order below. |
| Input path | Read one UTF-8 CSV with the exact schema in document 03; do not modify it. |
| `--date` | Required business date in strict `YYYY-MM-DD` form. |
| `--db` | Local SQLite database path. |
| `--report` | Destination for the one success JSON report. |

## Task and status contract

| Sequence | Task | Responsibility | Success condition |
|---:|---|---|---|
| 1 | `validate_csv` | Read and validate the file; partition valid and invalid rows. | Header is exact; every row is classified. |
| 2 | `load_sqlite` | Transactionally store run/task records, accepted orders, and quarantine rows. | All per-run rows and counts are persisted. |
| 3 | `write_report` | Write the deterministic JSON summary. | One report is written at the requested path. |

Task statuses are `pending`, `running`, `success`, `failed`, or `skipped`. On success, all three are `success`. If a task fails, it is `failed`; every later task is `skipped`. The run status is `success` only if all three tasks succeed. Persist task sequence, name, status, and attempt count.

The CSV reader is injectable for tests. A test-injected transient read error causes exactly one immediate retry (maximum two read attempts). If the second read fails, `validate_csv` fails. Do not retry validation errors, SQLite errors, or report errors.

## Run ID and replay

Compute SHA-256 over the source file's raw bytes. Form the run ID as `df-<business date without hyphens>-<first 12 lowercase checksum hex characters>`. For example, the format for the demo is `df-20261013-<12 lowercase hex characters>`. A repeated source checksum and business date reuse that logical run and do not add accepted or quarantined rows. Task statuses and counts reflect the latest attempt for that logical run.

## JSON success contract

The report uses UTF-8, two-space indentation, lexicographically sorted object keys, and a final newline. It contains no timestamp, duration, random value, absolute path, or machine-specific field. `source` is the input basename. The `tasks` array always follows sequence 1–3.

```json
{
  "accepted_rows": 18,
  "business_date": "2026-10-13",
  "quarantined_rows": 2,
  "run_id": "df-20261013-<12 lowercase hex characters>",
  "source": "orders.csv",
  "source_rows": 20,
  "source_sha256": "<64 lowercase hex characters>",
  "status": "success",
  "tasks": [
    {"attempts": 1, "name": "validate_csv", "sequence": 1, "status": "success"},
    {"attempts": 1, "name": "load_sqlite", "sequence": 2, "status": "success"},
    {"attempts": 1, "name": "write_report", "sequence": 3, "status": "success"}
  ]
}
```

The accepted/quarantined counts must equal the corresponding SQLite counts for this `run_id`. No report is written if an earlier task fails; the failed/skipped statuses remain in SQLite once a run ID exists.

## Errors and reset

- Invalid arguments, invalid date, missing/unreadable input, or invalid header: exit 2 and write `DATAFLOW_INPUT_ERROR: <reason>` to stderr.
- A task failure after run setup: exit 1 and write `DATAFLOW_TASK_FAILED: <task_name>` to stderr.
- If the source cannot be read to compute its checksum, no run ID can be established; return the input error without creating a run record.
- Reset is a separate `reset` action and must require `--confirm-reset`; a request without the flag changes no files.

[Back to README](../README.md)
