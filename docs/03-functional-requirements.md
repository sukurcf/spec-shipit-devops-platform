# 03 — Functional requirements

The following five requirements are Must scope. IDs are used by the test cases in [document 09](09-testing-strategy-and-test-cases.md).

| ID | Priority | Requirement and acceptance criteria |
|---|---|---|
| FR-DF-01 | Must | **Given** a UTF-8 CSV with the exact header `order_id,customer_id,order_date,amount` and no more than 500 data rows, **when** task 1 validates it, **then** each valid row is accepted and each invalid row is quarantined with one reason code. A file-level read or header error fails validation; no rows are loaded and later tasks are skipped. |
| FR-DF-02 | Must | **Given** a readable source and valid business date, **when** the runner starts, **then** it records one stable run ID and exactly these tasks in order: `validate_csv`, `load_sqlite`, `write_report`. Each task records status and attempts. A failure marks later tasks `skipped`. A transient read error injected by a test is retried once; no other failure is retried. |
| FR-DF-03 | Must | **Given** a successful validation, **when** task 2 loads results, **then** accepted orders, quarantined rows, run metadata, and task statuses are stored in one local SQLite database. The same source SHA-256 and business date use the same run ID and do not add duplicate rows. |
| FR-DF-04 | Must | **Given** tasks 1 and 2 succeeded, **when** task 3 completes, **then** it writes one UTF-8 JSON report at `--report` with source counts, stable run ID, overall status, and all three task names in order. The demo reports 20 source, 18 accepted, and 2 quarantined rows. Repeating the same command produces byte-identical report content. |
| FR-DF-05 | Must | **Given** a prepared local implementation checkout, **when** the student runs the command in document 08, **then** the runner uses only local files and exits 0 on success. Input/argument errors exit 2 with `DATAFLOW_INPUT_ERROR: <reason>`; task failures exit 1 with `DATAFLOW_TASK_FAILED: <task_name>`. Failed tasks and skipped downstream tasks remain recorded in SQLite when a run ID has been established. |

## Business and data rules

| ID | Rule |
|---|---|
| BR-01 | Header names and order are exact; files contain at most 500 data rows. |
| BR-02 | `order_id` and `customer_id` are non-empty after trimming. A blank order ID uses `missing_order_id`; a blank customer ID uses `missing_customer_id`. `order_id` is unique within one source; a later duplicate uses `duplicate_order_id` and the first occurrence is retained. |
| BR-03 | `order_date` is a real calendar date in strict `YYYY-MM-DD` form; an invalid value uses `invalid_order_date`. |
| BR-04 | `amount` is a finite decimal, non-negative, with at most two fractional digits. Use `invalid_amount` for nonnumeric or excess-precision values and `negative_amount` for a negative value. Store accepted money as integer cents. |
| BR-05 | The canonical fixture has 20 data rows: rows 1–18 are valid; row 19 has a blank `customer_id`; row 20 has `amount=-1.00`. Expected result: 18 accepted and 2 quarantined. |

Row-level invalids do not fail the run. A missing file, unreadable CSV, invalid header, or invalid business date is a run/task error, not a row quarantine.

[Back to README](../README.md)
