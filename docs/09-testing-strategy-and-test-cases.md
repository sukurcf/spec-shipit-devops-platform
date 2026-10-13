# 09 — Testing strategy and test cases

Use pytest with local fictional files and a temporary SQLite database. Tests that exercise read failures inject them through the reader seam. After dependencies are installed, all tests must pass with network access disabled. Persistence acceptance: after a test process exits, reopen the SQLite file and verify its rows and task statuses; replaying the same file/date leaves the stored counts unchanged. The named `dataflow` package target is at least 80% line and 70% branch coverage. The table defines 20 distinct Must cases.

| ID | Input and action | Expected result | Priority / trace |
|---|---|---|---|
| TC-DF-001 | Run the canonical 20-row fixture with the documented demo arguments. | Exit 0; 18 accepted, 2 quarantined; report and task order match document 08. | Must; FR-DF-01–05 |
| TC-DF-002 | Run a valid CSV with exactly 500 distinct valid rows. | Exit 0; 500 accepted, 0 quarantined. | Must; FR-DF-01 |
| TC-DF-003 | Run a valid CSV with 501 data rows. | Input validation fails; no load or report task runs. | Must; FR-DF-01, FR-DF-02 |
| TC-DF-004 | Use a header whose first two fields are reversed. | Exit 2; validation fails; load/report are skipped and no order is stored. | Must; FR-DF-01, FR-DF-02, FR-DF-05 |
| TC-DF-005 | Make the customer ID cell empty in one otherwise valid row. | That row is quarantined as missing_customer_id; other valid rows load. | Must; FR-DF-01, FR-DF-03 |
| TC-DF-006 | Make the order ID cell empty in one otherwise valid row. | That row is quarantined as missing_order_id; it is not inserted. | Must; FR-DF-01, FR-DF-03 |
| TC-DF-007 | Write one order date as slash-formatted text, 10/12/2026. | That row is quarantined as invalid_order_date. | Must; FR-DF-01 |
| TC-DF-008 | Put nonnumeric letters, abc, in one amount field. | That row is quarantined as invalid_amount. | Must; FR-DF-01 |
| TC-DF-009 | Put a negative amount, -1.00, in one row. | That row is quarantined as negative_amount; accepted amounts remain non-negative cents. | Must; FR-DF-01, FR-DF-03 |
| TC-DF-010 | Put a valid O-001 row before a second row with the same order ID. | First row is accepted; later duplicate is quarantined as duplicate_order_id; one O-001 exists for the run. | Must; FR-DF-01, FR-DF-03 |
| TC-DF-011 | Use a file containing the required header and no data rows. | Exit 0; 0 accepted, 0 quarantined; all three tasks succeed. | Must; FR-DF-01, FR-DF-02, FR-DF-04 |
| TC-DF-012 | Use an input path that does not exist. | Exit 2 with DATAFLOW_INPUT_ERROR; no run ID or data rows are created. | Must; FR-DF-05 |
| TC-DF-013 | Inject one transient read error, then return the canonical CSV. | Exit 0; validation uses exactly two attempts; counts remain 18/2. | Must; FR-DF-02, FR-DF-03 |
| TC-DF-014 | Inject a read error on both allowed attempts. | Validation fails after exactly two attempts; load/report are skipped; no order rows are loaded. | Must; FR-DF-02, FR-DF-03 |
| TC-DF-015 | Inject a SQLite load failure after validation. | Load task fails; its row transaction rolls back; report is skipped; failure is recorded. | Must; FR-DF-02, FR-DF-03 |
| TC-DF-016 | Run the same source bytes and business date twice against one database. | Same run ID; per-run counts stay 18 accepted and 2 quarantined; no duplicate rows. | Must; FR-DF-03 |
| TC-DF-017 | Compute run IDs for the same source bytes with business dates 2026-10-13 and 2026-10-14. | Run IDs differ because the business date is part of the identity. | Must; FR-DF-02, FR-DF-03 |
| TC-DF-018 | Inspect SQLite after a successful fixture run. | Exactly three task records exist in sequence 1–3 with the required names and success status. | Must; FR-DF-02 |
| TC-DF-019 | Compare report counts with SQLite rows for its run ID. | Report says 18/2 and matches 18 orders plus 2 quarantined_orders rows. | Must; FR-DF-03, FR-DF-04 |
| TC-DF-020 | Run the same successful command twice and compare report bytes. | The JSON bytes are identical; no timestamp or machine-specific path changes. | Must; FR-DF-03, FR-DF-04 |

### Traceability

| Requirement | Cases |
|---|---|
| FR-DF-01 | TC-DF-001–011 |
| FR-DF-02 | TC-DF-001, 003–004, 011, 013–018 |
| FR-DF-03 | TC-DF-001, 005–006, 009–010, 013–019 |
| FR-DF-04 | TC-DF-001, 011, 019–020 |
| FR-DF-05 | TC-DF-001, 004, 012 |

[Back to README](../README.md)
