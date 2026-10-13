# 10 — Development and CI quality

## Git workflow

Keep `main` releasable. Make small changes on a short-lived branch, open a pull request, review the diff and test output, then merge when the single CI job passes. Commit messages describe the change. Do not commit `.local/`, generated databases, reports, secrets, or real personal data.

## One CI job

Run a single job on **Python 3.12.x only**. This is one environment, not a version matrix.

| Order | Check | Pass condition |
|---:|---|---|
| 1 | Install the locked development dependencies | Lockfile resolves successfully |
| 2 | Ruff 0.16.x lint | No lint errors |
| 3 | pytest 9.1.x with pytest-cov 7.1.x | All 20 named cases pass; `dataflow` line coverage ≥80% and branch coverage ≥70% |

CI may download dependencies. Local tests and the demo run offline once the pinned dependencies are installed. This specification describes the job; it does not include workflow configuration files.

## Configuration and Definition of Done

| Setting | Required value |
|---|---|
| Input, business date, database, report | Supplied by the documented CLI arguments |
| Runtime environment variables | None required |
| Runtime network access / listening port | None |
| Local state | `.local/` in the student's implementation repository |

The work is done when all five FRs pass, the fixture produces 18/2, task order and stop behavior are observable, retry and replay tests pass, report counts match SQLite, coverage targets pass, the exact demo command works from a clean checkout after dependency setup, and the student can explain the implementation.

[Back to README](../README.md)
