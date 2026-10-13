# 07 — Data model

## Source contract

One CSV data row represents one order. Header order is exact. The canonical fixture has 20 rows; dates below are `2026-10-12`, and all amounts are `10.00` except the final invalid row.

| Data row(s) | `order_id` | `customer_id` | `order_date` | `amount` | Result |
|---|---|---|---|---:|---|
| 1–18 | `O-001` through `O-018` | Matching `C-001` through `C-018` | `2026-10-12` | `10.00` | Accepted |
| 19 | `O-019` | Empty | `2026-10-12` | `29.00` | Quarantined: `missing_customer_id` |
| 20 | `O-020` | `C-020` | `2026-10-12` | `-1.00` | Quarantined: `negative_amount` |

Other test fixtures may vary one field at a time. IDs are fictional. The demo business date is `2026-10-13`; it is run metadata, not an order-date filter.

## SQLite records

All tables are in one local SQLite file. `run_id` is `df-<YYYYMMDD>-<first 12 lowercase SHA-256 hex characters>`.

| Table | Grain and key | Required columns and rules |
|---|---|---|
| `pipeline_runs` | One logical run; primary key `run_id` | `business_date TEXT`, `source_sha256 TEXT`, `status TEXT`, `source_rows INTEGER`, `accepted_rows INTEGER`, `quarantined_rows INTEGER` |
| `task_runs` | One task in one run; primary key `(run_id, sequence)` | `task_name TEXT`, `status TEXT`, `attempts INTEGER`; sequence is 1, 2, or 3 |
| `orders` | One accepted order per logical run; primary key `(run_id, order_id)` | `customer_id TEXT`, `order_date TEXT`, `amount_cents INTEGER`; `amount_cents >= 0` |
| `quarantined_orders` | One rejected source row per logical run; primary key `(run_id, source_row_number)` | Nullable `order_id`, `reason_code TEXT`, original field values as JSON text |

`pipeline_runs` and `task_runs` record run/task outcomes. For a successful fixture run, the per-run `orders` count is 18 and `quarantined_orders` count is 2. The composite row keys make a same-checksum, same-date replay non-duplicating. A report count always refers to its `run_id`, not unrelated runs in the same database.

[Back to README](../README.md)
