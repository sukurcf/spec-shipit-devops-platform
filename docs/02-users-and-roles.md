# 02 — Users and roles

## Roles and personas

| Role | Persona | Goal | Owns |
|---|---|---|---|
| Student Data Engineer | Fresher practicing local batch work | Run the fixture, inspect outcomes, and explain the design | Implementation, tests, and files under the student's `.local/` directory |
| Trainer | Reviewer and assessor | Verify behavior against this specification | Acceptance decisions and feedback |

Only fictional order data is used. The trainer inspects evidence supplied by the student; the trainer does not operate the student's local database.

## User stories

| ID | Story |
|---|---|
| US-01 | As a student, I want one documented command to run the three fixed tasks so that I can reproduce the demo locally. |
| US-02 | As a student, I want invalid rows stored with reasons so that accepted data is not silently mixed with rejected data. |
| US-03 | As a trainer, I want the run ID, task statuses, and row counts to be inspectable so that I can check ordering, failure handling, and idempotency. |

## Ownership and access

| Artifact | Student/runner | Trainer |
|---|---|---|
| Input fixture | Reads; does not modify | Reads during review |
| SQLite file and JSON report | Creates and inspects locally | Reviews submitted evidence |
| Test results and implementation | Creates and explains | Reviews against the rubric |
| Reset of local state | Student-initiated and explicitly confirmed | No direct write access |

The runner reads the named input file and writes only to the explicitly supplied database and report paths. No credentials or real personal data are needed.

[Back to README](../README.md)
