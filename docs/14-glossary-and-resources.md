# 14 — Glossary and resources

## Glossary

| Term | Meaning |
|---|---|
| Batch | A finite set of records processed together. |
| Pipeline | Ordered data-processing steps that produce a result. |
| Task | One named unit of work in the pipeline. |
| Dependency order | The required sequence in which tasks may run. |
| CSV | Comma-separated text records with a header row. |
| Header | The ordered column names at the top of a CSV. |
| Data contract | Agreed field names, types, and validation rules. |
| Validation | Checking data against the contract. |
| Quarantine | Separate storage for rejected rows and their reasons. |
| SQLite | A relational database stored in a local file. |
| Transaction | A group of database changes committed together or rolled back. |
| Primary key | A column or column group that uniquely identifies a row. |
| Composite key | A primary key made from multiple columns. |
| Idempotency | Repeating the same operation without duplicating its result. |
| Checksum | A digest used to identify file content. |
| Run ID | Stable identifier for one logical source/date run. |
| JSON | A text format for structured data and reports. |
| ISO date | A calendar date written as `YYYY-MM-DD`. |
| Unit test | A focused automated check of a behavior. |
| Branch coverage | The fraction of decision outcomes exercised by tests. |

## Five recommended official readings

1. [Python 3.12 CSV module](https://docs.python.org/3.12/library/csv.html) — Reading and writing delimited records.
2. [Python 3.12 SQLite interface](https://docs.python.org/3.12/library/sqlite3.html) — Local databases and transactions.
3. [Python 3.12 JSON and Decimal modules](https://docs.python.org/3.12/library/json.html) and [Decimal arithmetic](https://docs.python.org/3.12/library/decimal.html) — Stable reports and exact decimal parsing.
4. [pytest documentation](https://docs.pytest.org/en/stable/) — Writing and running tests.
5. [uv documentation](https://docs.astral.sh/uv/) and [Ruff documentation](https://docs.astral.sh/ruff/) — Managing the environment and linting.

Additional official references: [Python argparse](https://docs.python.org/3.12/library/argparse.html), [Python hashlib](https://docs.python.org/3.12/library/hashlib.html), [SQLite transactions](https://www.sqlite.org/lang_transaction.html), [pytest-cov](https://pytest-cov.readthedocs.io/en/latest/), and [GitHub Actions](https://docs.github.com/en/actions).

[Back to README](../README.md)
