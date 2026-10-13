# 06 — Tech stack and setup

## Pinned cohort tool families

| Tool | Version family | Use |
|---|---|---|
| Python | 3.12.x | Runtime and standard library (`csv`, `sqlite3`, `json`, `argparse`, `hashlib`, `decimal`, `datetime`) |
| uv | 0.12.x | Reproducible environment and CLI invocation |
| pytest | 9.1.x | Required test runner |
| pytest-cov | 7.1.x | Measure the named package coverage target |
| Ruff | 0.16.x | Lint |

There are no runtime package dependencies. Use the standard-library `argparse` interface; Typer is not needed. Keep the student's dependency lock consistent with these existing cohort families.

## Local setup

Use Python 3.12.x, Git, and uv on macOS, Linux, or Windows 11 with WSL2. From the student's implementation repository, install its pinned development dependencies once with `uv sync --locked`. Dependency download is the only step that needs network access.

## Demo and outputs

From the implementation repository, run exactly:

```text
uv run dataflow run fixtures/orders.csv --date 2026-10-13 --db .local/dataflow.db --report .local/report.json
```

No server starts and no port is opened or bound to `127.0.0.1`; this CLI process starts and exits with the demo command, so no separate service start/stop commands apply. The CSV is read-only. The database and report live under `.local/`; the quarantine rows are in SQLite. The successful fixture result is 18 accepted and 2 quarantined rows with all three tasks successful. Re-running the same file and date leaves the per-run row counts unchanged.

The runner uses local files only and has no runtime network calls. Tests and the demo pass offline after dependency installation.

## Reset and hardware

Do not delete persistent state implicitly. Any reset action is separate from `run` and requires the explicit confirmation flag: `uv run dataflow reset --db .local/dataflow.db --confirm-reset`. Without that flag, a reset request fails and leaves files unchanged.

Proposed minimum: 2 CPU cores, 4 GB RAM, and 1 GB free disk. No ports or additional memory allocation are required.

[Back to README](../README.md)
