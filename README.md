# DataFlow

DataFlow is a five-day, local Python task runner specification for one fictional orders batch. The complete brief is designed for about a one-hour read and stays under 60 KB. Students implement it in their own repository; this repository contains Markdown specifications only.

## Quick facts

| Item | Contract |
|---|---|
| Target role | Junior Data Engineer |
| Duration | 5 days × 6 hours = 30 hours |
| Input | One UTF-8 `orders.csv`, 20 fictional rows in the demo |
| Result | 18 accepted rows, 2 quarantined rows in one SQLite database |
| Interface | One local CLI; no listening port |
| Demo | `uv run dataflow run fixtures/orders.csv --date 2026-10-13 --db .local/dataflow.db --report .local/report.json` |
| Network | No runtime network access; install pinned dependencies before the demo |
| Specification size | 15 Markdown files; no implementation code |

## Demo contract

Run the command above from the student's implementation repository. The successful run records exactly three tasks in this order: `validate_csv`, `load_sqlite`, `write_report`. Its deterministic JSON report has `status=success`, `source_rows=20`, `accepted_rows=18`, and `quarantined_rows=2`; every task status is `success`. The database contains 18 accepted and 2 quarantined rows for that run. Repeating the same file and business date does not add rows.

The run ID is stable for the source checksum and business date. The `.local/` directory holds the database and report and is excluded from Git. The quarantine is stored in SQLite; no separate report or service is required.

## Scope

Build a fixed three-step runner with Python 3.12.x, standard-library CSV/SQLite/JSON handling, and pytest. Implement row validation, SQLite persistence, failure status recording, one retry for an injected transient read error, replay safety, and a deterministic summary. See [the task-runner contract](docs/08-platform-specification.md).

Out of scope: additional pipeline steps, a general scheduler, remote storage, network calls, listening ports, browser UI, and distributed execution.

## Five-day outline

| Day | Hours | Goal |
|---|---:|---|
| 1 | 6 | Setup, inspect the fixture, agree the data contract, create a thin slice and first test. |
| 2 | 6 | Implement CSV input and validation. |
| 3 | 6 | Implement SQLite persistence and reporting. |
| 4 | 6 | Finish failure handling, idempotency, tests, and coverage. |
| 5 | 6 | Run from a clean clone, fix defects, complete documentation and demo, explain design and tests. |

## Read the specification

| Document | Purpose |
|---|---|
| [01 Project overview](docs/01-project-overview.md) | Problem, outcome, scope, and job-skill mapping |
| [02 Users and roles](docs/02-users-and-roles.md) | Roles, ownership, and user stories |
| [03 Functional requirements](docs/03-functional-requirements.md) | Five Must requirements and data rules |
| [04 Non-functional requirements](docs/04-non-functional-requirements.md) | Local quality, privacy, and resource targets |
| [05 System architecture](docs/05-system-architecture.md) | Fixed task flow and design decisions |
| [06 Tech stack and setup](docs/06-tech-stack-and-setup.md) | Tool versions, setup, demo, and cleanup |
| [07 Data model](docs/07-data-model.md) | Fixture and SQLite records |
| [08 Task-runner specification](docs/08-platform-specification.md) | Exact CLI, task, retry, status, and JSON contracts |
| [09 Testing strategy and cases](docs/09-testing-strategy-and-test-cases.md) | 20 concrete Must test cases |
| [10 Development and CI quality](docs/10-devops-ci-cd-and-quality.md) | Git flow, one CI job, and Definition of Done |
| [11 Milestones and deliverables](docs/11-milestones-and-deliverables.md) | Five-day plan and five-minute demo |
| [12 Evaluation rubric](docs/12-evaluation-rubric.md) | 100-point grading and pass gates |
| [13 Interview preparation](docs/13-interview-preparation.md) | Project-specific questions and one-minute pitch |
| [14 Glossary and resources](docs/14-glossary-and-resources.md) | Terms and official learning links |

## Student work

Use Git branches and small pull requests. AI tools may support learning, debugging, and review; understand and disclose substantial assistance as directed by the trainer. The student implementation is separate from this Markdown-only specification repository.

## Change log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-10-02 | First release |
| 1.1 | 2026-10-02 | Backend/local DevOps only. Frontend/custom visualization and AWS deployment tasks/points are removed; effort moves to API contracts, metric/alert evidence, offline Compose/kind operations, and artifact validation. Original Python coverage and 45-test minimum remain. |
| 1.2 | 2026-10-13 | Replaces the six-week scope with five days / 30 hours for the local DataFlow runner. Grading now uses the 15–20-case target (20 specified cases), 80% line / 70% branch coverage, and the 100-point local-pipeline rubric; prior platform deliverables and the 45-test minimum no longer apply. |
