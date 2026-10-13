# 11 — Milestones and deliverables

The time budget is exactly five days × six hours = 30 hours. All setup, implementation, tests, documentation, and the final demo fit within this plan.

| Day | Hours | Goal and deliverable |
|---|---:|---|
| Day 1 | 6 | Read the short spec; set up Python/uv; inspect the fictional fixture; agree the CSV contract; create the first thin slice and one test (FR-DF-01, TC-DF-001). |
| Day 2 | 6 | Implement CSV input, header checks, row validation, and quarantine reasons (FR-DF-01, TC-DF-002–012). |
| Day 3 | 6 | Implement the three fixed tasks, SQLite schema/transaction, stable run ID, and JSON summary (FR-DF-02–04, TC-DF-016–020). |
| Day 4 | 6 | Add failure recording, downstream stop behavior, one retry, replay safety, and remaining tests; meet 80% line / 70% branch coverage (FR-DF-02–05, TC-DF-013–020). |
| Day 5 | 6 | Run from a clean clone; fix defects; finish README and demo; explain design and tests. No Must feature remains unfinished. |

## Five-minute demo

1. **0:00–0:45 — Inputs:** Show the fictional CSV header and identify the two intentional invalid rows.
2. **0:45–2:00 — Run:** Execute `uv run dataflow run fixtures/orders.csv --date 2026-10-13 --db .local/dataflow.db --report .local/report.json`.
3. **2:00–3:00 — Result:** Show the deterministic report, the task order, and 18 accepted / 2 quarantined rows.
4. **3:00–4:00 — Persistence:** Show that report counts match SQLite, then repeat the same command and confirm no duplicate rows.
5. **4:00–5:00 — Tests and explanation:** Show the retry/failure test and explain the run ID, transaction, and downstream-stop behavior.

The demo describes the expected student implementation. This specification repository does not contain or claim to have run that implementation.

[Back to README](../README.md)
