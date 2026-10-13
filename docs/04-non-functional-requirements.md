# 04 — Non-functional requirements

| ID | Requirement |
|---|---|
| NFR-DF-01 | Local portability: the prepared project runs on Python 3.12.x on macOS, Linux, or Windows 11 through WSL2. It uses files and one SQLite database; no port or runtime network access is needed. |
| NFR-DF-02 | Integrity: loading accepted and quarantined rows is transactional. A failed load leaves no partial row set for that task. Run/task failure status remains inspectable after the failure. |
| NFR-DF-03 | Determinism: run ID derives from source SHA-256 and business date. The report omits timestamps, durations, random identifiers, and absolute paths; identical inputs and arguments produce identical bytes. |
| NFR-DF-04 | Privacy: fixtures contain fictional values only. Do not store credentials, real personal data, or secrets in Git or reports. |
| NFR-DF-05 | Maintainability: keep the implementation small and testable. The named `dataflow` Python package targets at least 80% line coverage and 70% branch coverage. No 100% coverage target applies. |
| NFR-DF-06 | Resource target, proposed and not measured: process up to 500 CSV rows in under 5 seconds on a 2-core laptop with 4 GB RAM and 1 GB free disk, excluding initial Python startup and dependency installation. Timing is not a CI gate. |

Tests use local fictional files and temporary local paths. After pinned dependencies are installed, the demo and tests do not need internet access. Hardware figures are proposed course targets, not benchmark claims.

[Back to README](../README.md)
