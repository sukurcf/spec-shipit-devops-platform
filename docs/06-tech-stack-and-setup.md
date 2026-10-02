# Tech stack and setup

Purpose: This document pins the ShipIt tools, explains setup choices, and gives laptop profiles that keep the Must scope reproducible.

## Reference versions and mandatory stack

| Category | Tool | Reference version | Purpose | Why this choice |
|---|---|---|---|---|
| Language | Python | 3.12.x | Build the FastAPI ShortLink app. | It is the cohort baseline and has long security support. |
| Package manager | uv | 0.12.x | Manage Python dependencies and `uv.lock`. | It is fast and gives reproducible installs. |
| Web framework | FastAPI | 0.142.x | Implement ShortLink endpoints. | It has clear OpenAPI support and is common in Python roles. |
| Settings | Pydantic and pydantic-settings | 2.13.x and 2.15.x | Validate request models and environment configuration. | It gives explicit validation errors for `DATABASE_URL` and URLs. |
| Database | PostgreSQL | 18.x | Store links, redirect events, and app data. | It is the default relational database for DevOps interviews. |
| Database driver | psycopg | 3.3.x | Connect FastAPI to PostgreSQL. | It is the current PostgreSQL driver line for Python. |
| ORM and migrations | SQLAlchemy and Alembic | 2.1.x and 1.20.x | Model tables and apply schema changes. | They teach durable migration practice. |
| App server | Uvicorn and Gunicorn | 0.54.x and 26.x | Serve the FastAPI app in containers. | They separate development and production serving concerns. |
| Quality | Ruff, mypy, pytest, pytest-cov | 0.16.x, 2.4.x, 9.1.x, 7.1.x | Lint, type-check, test, and measure coverage. | They support the 80% line and 70% branch gates. |
| Git hooks | pre-commit | 4.6.x | Run local checks before commits. | It catches formatting and secret issues early. |
| Containers | Docker Engine and Docker Compose | 29.x and v5.x | Build images and run local app plus PostgreSQL. | They match common entry-level DevOps job requirements. |
| Container lint | hadolint | 2.15.x | Check Dockerfile quality. | It catches root user and package-cache mistakes. |
| Shell lint | ShellCheck | 0.11.x | Check operational scripts. | It enforces safe Bash for `bootstrap.sh` and backup scripts. |
| Kubernetes local | kind | 0.33.x | Run Kubernetes locally. | It uses Kubernetes 1.37.x and needs no cloud account. |
| Kubernetes CLI | kubectl | 1.37.x | Inspect cluster resources. | It matches the kind Kubernetes minor line. |
| Package deployment | Helm | 4.x | Package the `shortlink` chart and run rollback. | Helm 4.x is the cohort reference line. |
| IaC | Terraform | 1.16.x | Create kind and install platform add-ons. | It is the default job-market keyword. |
| Terraform providers | `hashicorp/helm`, `tehcyx/kind` | 3.3.x, 0.11.x | Install add-ons and create kind. | They keep the IaC scope small. |
| Gateway | Gateway API and Envoy Gateway | 1.6.x and 1.9.x | Route `.localhost` traffic. | ingress-nginx is not allowed. |
| Monitoring | kube-prometheus-stack | 91.x | Install Prometheus, Alertmanager, and Grafana. | It is the standard local observability stack. |
| Optional operations console | Grafana OSS | 13.x | Inspect unmodified built-in tooling in standard only. | It is not a frontend implementation, required evidence source, or custom dashboard/panel task. |
| Alert sink | Mailpit | 1.31.x | Receive Alertmanager e-mail locally. | MailHog is unmaintained. |
| Security scan | Trivy and Gitleaks | 0.75.x and 8.30.x | Scan images, manifests, and secrets. | They enforce NFR-SEC-02 and NFR-SEC-03. |
| Manifest schema | kubeconform | 0.8.x | Validate rendered Kubernetes resources. | It detects invalid API versions before apply. |
| Load test | k6 | 2.x | Prove redirect latency and rolling updates. | It creates repeatable performance evidence. |
| Optional cache | Valkey or Redis | Valkey 9.x or Redis 8.x | Add cache-aside redirects. | Valkey is preferred when licence concerns matter. |
| Optional GitOps | Argo CD | 3.5.x | Watch `deploy/` after Must scope. | It teaches pull-based delivery. |
| Optional logs | Loki and Grafana Alloy | 3.7.x and 1.20.x | Centralize local logs. | Alloy replaces deprecated Promtail. |
| Optional scaling | metrics-server and Calico | 0.9.x and 3.33.x | HPA and NetworkPolicy enforcement. | kind needs explicit add-ons for these features. |
| Optional CI comparison | Jenkins LTS | 2.568.x | Explain GitHub Actions versus Jenkins. | Jenkins appears in many Indian DevOps job descriptions. |

## Allowed alternatives that require an ADR

| Alternative | ADR question | Boundary |
|---|---|---|
| OpenTofu 1.13.x instead of Terraform 1.16.x | Why does this portfolio choose OpenTofu over Terraform? | The ADR MUST compare state, providers, job keywords, and licence concerns. |
| Redis 8.x instead of Valkey 9.x | Why is Redis acceptable for the optional cache? | The ADR MUST mention licence and that caching is Should scope. |
| Cilium instead of Calico | Why is Cilium used for NetworkPolicy on kind? | The ADR MUST prove negative database connectivity tests. |
| SOPS instead of Sealed Secrets | How are local secrets encrypted and decrypted? | The ADR MUST show that plain passwords never enter Git. |

## Not allowed

| Tool or pattern | Reason |
|---|---|
| ingress-nginx | The cohort uses Gateway API with Envoy Gateway because ingress-nginx is retired. |
| Bitnami images or charts | The free catalog changed in 2025, and Must PostgreSQL uses a direct StatefulSet. |
| Docker image tags with `latest` | They break reproducibility and security evidence. |
| Committed `.env` files with secrets | They violate NFR-SEC-03 and fail Gitleaks. |
| Promtail | It is deprecated. Use Grafana Alloy for optional logs. |
| Public/cloud deployment or managed Kubernetes exercises | All implementation scope is local; AWS concepts are interview theory only. |
| A PostgreSQL operator for Must scope | The brief requires a StatefulSet, PVC, and Service only. |
| Workflow YAML copied from a tutorial without explanation | Students must explain every gate in viva. |
| Frontend, helper UI, custom dashboard/panel/visualization/Geomap authoring or exports | No such Must, Should, Could, alternative, or bonus work is allowed. |

## Hardware profiles

| Profile | Laptop target | Cluster shape | Enabled services | Proposed limits |
|---|---|---|---|---|
| Lite | 8 GB RAM, 4 cores, 20 GB free disk | Stage 1 Compose; stage 2 strictly 1-node kind | One active ShortLink/database environment, Envoy, Prometheus/Alertmanager, Mailpit | Active app 1 replica at 256 MiB; PostgreSQL 512 MiB; rollout surge 1; retention 6 hours; Grafana/Loki/Argo CD off. |
| Standard | 16 GB RAM, 4 or more cores, 30 GB free disk | 1-node kind Must; 3-node Should after separate pre-check | Same core services; one active environment in the canonical profile; optional add-ons separately budgeted | Dev 1/prod 2 app replicas at 512 MiB/pod; active PostgreSQL 1 GiB; surge 1; retention 24 hours. |

Compose and kind MUST NOT run concurrently in the canonical local profile. Both dev/prod definitions exist in kind, but inactive app/PostgreSQL workloads have 0 replicas and retained PVCs. Standard-only HPA/multi-node/optional add-ons require a separate resource pre-check; they never expand the 8 GB profile. These budgets are proposed planning caps, not measured results:

| Component | Lite budget | Standard budget |
|---|---:|---:|
| kind node overhead | 700 MiB | 1,200 MiB |
| Envoy Gateway controller and proxy | 192 MiB | 384 MiB |
| Prometheus | 640 MiB | 1,100 MiB |
| Grafana | Disabled | Disabled by default; 320 MiB if optionally enabled |
| Alertmanager | 96 MiB | 160 MiB |
| Prometheus Operator | 192 MiB | 256 MiB |
| kube-state-metrics | 96 MiB | 160 MiB |
| node-exporter | 64 MiB | 96 MiB |
| Mailpit | 64 MiB | 128 MiB |
| Active ShortLink app pods | 256 MiB total | 1,024 MiB total (prod) |
| One active PostgreSQL pod | 512 MiB | 1,024 MiB |
| **Proposed core steady envelope** | **2,812 MiB** | **5,532 MiB** (optional Grafana/add-ons excluded) |
| Additional rolling-update app surge reserve | 256 MiB | 512 MiB |

Set explicit component resource requests/limits in Compose/Helm values where supported. Compose app/PostgreSQL caps are 256/512 MiB in lite and 512/1,024 MiB in standard. For kind app CPU, propose request 100m/limit 500m (lite) or request 250m/limit 1000m (standard). Node overhead is an estimate, not a pod limit. Reserve remaining host/WSL capacity for Docker, Kubernetes, migrations, Helm hooks, and rollout peaks. The trainer MUST record startup/steady/load/surge peaks and adjust the local profile through a clarification if it cannot run reliably; swap is not proof of adequate RAM.

## Windows 11 with WSL2 checklist

| Step | Check |
|---|---|
| 1 | Install WSL2 with Ubuntu 24.04. |
| 2 | Set `.wslconfig` for an 8 GB laptop to `memory=4GB` and `swap=4GB`. |
| 3 | Set `.wslconfig` for a 16 GB laptop to `memory=8GB` and `swap=4GB`. |
| 4 | Install Docker Desktop with the WSL2 backend, or an approved Docker Engine setup in WSL2. |
| 5 | Install Bash, make, Git, Python 3.12.x, uv, and VS Code Remote WSL; add kubectl 1.37.x, kind 0.33.x, Helm 4.x, Terraform 1.16.x, and k6 2.x before stage 2/drills. |
| 6 | Confirm Docker can run Linux containers from Ubuntu. |
| 7 | Run the future `bootstrap.sh` check mode before creating the cluster. |
| 8 | If RAM is below 16 GB, select the lite profile. |

## macOS checklist

| Step | Check |
|---|---|
| 1 | Use Apple Silicon or Intel macOS with Docker Desktop or a compatible local Docker engine. |
| 2 | Allocate at least 4 GB memory to Docker for lite, or 8 GB for standard. |
| 3 | Install Command Line Tools/make, Bash, Git, Python 3.12.x, uv, and VS Code; add the approved kubectl/kind/Helm/Terraform/k6 families before stage 2/drills. |
| 4 | Verify loopback requests with explicit `.localhost` Host headers; no hosts-file edit is needed. |
| 5 | Use lite mode if Docker memory is below 8 GB. |

## Linux checklist

| Step | Check |
|---|---|
| 1 | Use Ubuntu 24.04 or a similar modern distribution. |
| 2 | Install Docker Engine 29.x and add the user to the Docker group only if approved by the trainer. |
| 3 | Install Bash, make, Git, Python 3.12.x, uv, and an editor; add approved kubectl/kind/Helm/Terraform/k6 families before stage 2/drills. |
| 4 | Confirm cgroup v2 and container networking work before week 1 ends. |
| 5 | Use `127.0.0.1` and an explicit Host header for kind; do not depend on external DNS or follow redirects. |

## Local operation contract

FR-LOCAL-01 defines commands students MUST implement in the future student repository. Nothing here is a supplied script or a claim that an application has run.

### Preparation, secrets, and offline artifacts

- Support Windows 11 WSL2 Ubuntu, macOS, and Linux with the same make entrypoints. Initial tool/Python/package/container/chart/provider downloads need internet. GitHub submission, hosted CI, scans, and GHCR publishing are separate connected exercises.
- Implement `make local-prepare STAGE=compose` for initial Python wheels/base/PostgreSQL images and ignored credentials. Before stage 2, `make local-prepare STAGE=kind` also caches the kind node image, add-on/hook images, chart archives/dependencies/CRDs, Terraform providers, schema files, and old/new/rollback app images. Cache in ignored `.local/cache/` or Docker's image store. Store fictional recorded API/Prometheus/alert fixtures in student tests. Stage 1 does not require Kubernetes tools, hosted CI, or registry credentials.
- Terraform MUST use the prepared local provider mirror/plugin cache and chart archives, with local modules and locked providers. No Helm repository refresh, provider download, image registry pull, or GitHub request occurs during offline startup. Load all required add-on/hook images into kind as well as the app. Missing cache entries fail clearly rather than silently downloading.
- Generate local development credentials into ignored `.local/compose-secrets.env`, `.local/dev-secrets.env`, and `.local/prod-secrets.env`; restrict file permissions. Commit only names/placeholders in `.env.example`. `scripts/create-secrets.sh` consumes the selected file for Kubernetes Secrets. No paid account, cloud resource, public hostname, or API key is required for Must local operation.

### One start interface and one stop interface

| Command to implement | Required behavior |
|---|---|
| `make local-start` | Defaults `STAGE=compose PROFILE=lite ENV=dev`. Pre-check tools, Docker, ports, secrets, cache, and disk/RAM. Build/use a local app image without pulling; start app + PostgreSQL, run migrations/fictional seed once, wait for health/readiness, print `local-ready stage=compose env=dev`. |
| `make local-start STAGE=kind PROFILE=lite ENV=prod` | Only after Compose is stopped. Create or resume one-node `shipit-local` kind through Terraform/cached add-ons. Build the app from cached dependencies, execute `kind load docker-image shortlink:local-<short-sha> --name shipit-local`, use that exact tag with `imagePullPolicy: IfNotPresent` (or `Never`). Install both definitions; activate only the selected database, migrate/seed before app readiness, start loopback port-forwards, and print `local-ready stage=kind env=prod`. |
| `make local-stop STAGE=compose` | Default stop stage is Compose. Stop only project-owned services/processes without removing the named PostgreSQL volume; print `local-stopped stage=compose data-preserved`. |
| `make local-stop STAGE=kind` | Stop project-owned port-forwards, scale app/database workloads to 0 cleanly, then stop owned kind node containers without deleting them, local state, or data mounts. Print `local-stopped stage=kind data-preserved`. Restart resumes the existing node(s) before idempotent apply/Helm and restores the selected replica counts. |
| `make local-demo STAGE=compose` or `make local-demo STAGE=kind ENV=prod` | Check health/readiness, request the deterministic seeded redirect without following it, request one missing code, and print the exact summary below. kind uses the selected Host header and also saves metrics/alert evidence. |
| `make local-reset STAGE=compose CONFIRM=DELETE_LOCAL_DATA` (or `STAGE=kind`) | Only while the selected stage is stopped: explicitly delete its project-local data/cluster/state. Without confirmation exit 2 `local-reset-confirmation-required`. Do not delete other projects, dependency caches, or backups. |

Use SHA-derived immutable local tags such as `shortlink:local-abc1234`, never `latest`. A new rollout build gets a new tag; load good/bad/rollback tags before isolation. Registry-delivery values are separate from local values: hosted GHCR tags/digests and credentials are never default local inputs.

Compose PostgreSQL uses named volume `shipit-compose-postgres`. kind uses separate retained dev/prod PVCs backed by ignored `.local/kind-data/dev/` and `.local/kind-data/prod/` through documented node bind mounts/PVs. State, kubeconfig, PID records, and redacted evidence live under ignored `.local/`. Run one Alembic initialization step after database health and before app readiness, not concurrent migrations in app replicas. Stop/start retains link rows, PVC identity, and schema; seed only if absent and never overwrite links. A conflicting seed code fails clearly with `local-seed-conflict: Ab3kP91`. Stages have independent databases; backup/restore targets the explicit stage/environment.

Only one app environment runs at once in the canonical profiles. Switching kind `ENV` requires stop, then start the other environment. Both namespace/Secret/StatefulSet/PVC definitions remain; inactive app/database replica counts are 0. During selected-database initialization, app replicas may temporarily be 0 until the single migration/seed step finishes; restore the profile count before readiness succeeds. Stage 1 uses dev; invalid stage/environment combinations fail clearly. Monitoring/alerts select the active environment. Multi-node/HPA is standard-only after separate resource validation.

### Local ports and deterministic demo

All host-published ports/port-forwards MUST bind to `127.0.0.1`, including the kind API server. PostgreSQL and Mailpit SMTP remain internal; access PostgreSQL via Compose/kubectl exec, not a public host port.

| Service | Fixed host endpoint | Health/evidence |
|---|---|---|
| Compose API (stage 1) | `127.0.0.1:8080` | `/healthz` 200 with `status=ok`, `service=shortlink`; `/readyz` 200 with `status=ready`. |
| kind Gateway (stage 2) | `127.0.0.1:8081` | Same checks with Host `shortlink-dev.localhost` or `shortlink.localhost`; forward service port 80. |
| kind Kubernetes API | `127.0.0.1:6445` | kubectl cluster/node readiness. |
| Prometheus | `127.0.0.1:9090` | `/-/ready` 200; `/api/v1/query` and `/api/v1/alerts` return success JSON. |
| Alertmanager | `127.0.0.1:9093` | `/-/ready` 200; `/api/v2/alerts` saves alert identity/labels. |
| Mailpit HTTP API | `127.0.0.1:8025` | Inspect `/api/v1/messages` for local alert e-mail; SMTP is internal `mailpit.monitoring:1025`. |
| Optional built-in Grafana | `127.0.0.1:3000` | Standard operations-only, disabled by default; no custom dashboard/panel or mandatory UI demo. |

Seed one fictional active link `Ab3kP91` → `https://example.in/careers`, created at `2026-10-02T00:00:00Z`, with no expiry/deletion. Seed code is a training fixture, not a replacement for cryptographically generated codes in POST. After readiness, `make local-demo` executes equivalent API checks at the selected loopback endpoint (kind with explicit Host) and prints:

```text
health=200 ready=200
seed_code=Ab3kP91 redirect_status=302 location=https://example.in/careers
missing_status=404 error=short-code-not-found
```

Use curl without `-L`, or a Python client with redirect following disabled; never contact the fictional Location. Capture the successful `shortlink_redirects_total{status="redirected"}` series before/after: the seed redirect increases it by exactly 1; health/readiness/missing-code requests do not increase that series. Prometheus evidence uses document 08 queries; repeating the demo adds no seed row.

TC-LOCAL-001 through TC-LOCAL-004 run with external egress denied but loopback/container/cluster networking allowed after preparation. Invalid stage/profile/input, unavailable Docker, missing secrets/cache, or occupied ports fail before side effects with exit 2 and named errors (`profile-invalid`, `local-dependency-unavailable: docker`, `secret-file-missing`, `local-artifact-missing: <path>`, `local-port-in-use: <port>`). A PostgreSQL health deadline fails with exit 3 `local-readiness-timeout: postgres`; a running app keeps `/healthz` 200 but returns `/readyz` 503 with `status=not_ready`, `dependency=postgresql`, `error=database-not-ready`. Do not download, erase data, or fall back to GHCR on failure.

## Bootstrap sequence

| Order | Action |
|---:|---|
| 1 | Run `bootstrap.sh --check-only`, select lite/standard, prepare downloads/cache and ignored secrets. |
| 2 | Run `make local-start` for Compose; verify the seed demo, persistence, and stop. |
| 3 | Run `make local-start STAGE=kind PROFILE=lite ENV=dev`; use local Terraform artifacts and load app/add-on images. |
| 4 | Create each environment's Secret from its ignored file; keep the inactive app/database at 0. |
| 5 | Verify dev API/evidence; stop and start `ENV=prod` sequentially, then run the same checks. |
| 6 | Run local acceptance, record metrics/alerts through APIs, and use the matching stage stop command. |

## Trainer pre-check

- Validate the lite profile on an 8 GB laptop before week 1.
- First verify Compose app/PostgreSQL, seed/migrations, health, stop/start, and failure cases.
- Exercise dev and prod kind sequentially with one active app/database; never require both to run simultaneously on lite.
- Confirm `memory=4GB` and `swap=4GB` work for WSL2 lite.
- Create a 1-node kind 0.33.x cluster with Kubernetes 1.37.x.
- Create dev Secrets with `scripts/create-secrets.sh` from an ignored local file.
- Create prod Secrets with `scripts/create-secrets.sh` from an ignored local file.
- Install Envoy Gateway 1.9.x and kube-prometheus-stack 91.x.
- Confirm retention 6 hours, one active PostgreSQL pod, app 1 replica plus rollout surge capacity, and inactive workloads 0.
- Confirm Grafana, Loki, Argo CD, HPA, and multi-node work are absent in lite.
- Verify local image loading/pull policy and cached provider/chart/add-on setup with external egress denied.
- Run TC-LOCAL-001 through TC-LOCAL-004 for both stages and record exact demo, persistence, failure messages, and loopback binds.
- Record host/WSL memory (8 GB host: 4 GB memory/4 GB swap; 16 GB host: 8 GB memory/4 GB swap), disk, and observed peaks against proposed caps. This specification supplies no measured 8 GB claim.
- Record any trainer-only machine issue as a clarification issue.

## Free accounts needed

| Account | Required? | Purpose |
|---|---|---|
| GitHub | Must for submission/CI, not local startup | Student repository, pull requests, GitHub Actions, GHCR, Dependabot. |
| Docker Hub | Should | Pull base images if anonymous rate limits block setup. |
| Grafana Cloud | No | Not needed because Grafana runs locally. |
| AWS | No for any implementation scope | Interview theory only; no account or cloud deployment exercise. |

## Suggested learning order

| Order | Topic | Hours | Outcome |
|---:|---|---:|---|
| 1 | Linux processes, permissions, ports, and networking commands | 5 | Student can diagnose listening ports and failed processes. |
| 2 | Docker images, containers, layers, volumes, and Compose | 6 | Student can explain image versus container and persistent PostgreSQL data. |
| 3 | FastAPI health, readiness, metrics, and PostgreSQL basics | 5 | Student can run and test the ShortLink app. |
| 4 | GitHub Flow, Conventional Commits, and GitHub Actions gates | 4 | Student can open small PRs and read failed checks. |
| 5 | Kubernetes objects, probes, resources, Gateway API, and kind | 6 | Student can inspect pods, Services, StatefulSets, Gateway, and HTTPRoute. |
| 6 | Helm chart concepts and rollback | 3 | Student can explain chart, values, release, and revision. |
| 7 | Terraform state, plan, apply, variables, modules, and drift | 4 | Student can explain local state and why remote state maps to S3. |
| 8 | Prometheus HTTP API, PromQL, Alertmanager, RED, USE, SLI, SLO, and SLA | 5 | Student can save metric/alert JSON and explain results without visualization authoring. |
| 9 | Security scans, secrets, Dependabot, and vulnerability triage | 3 | Student can justify Trivy and Gitleaks failures. |
| 10 | Local operation acceptance and AWS interview theory | 4 | Student proves offline start/stop/persistence and maps concepts without cloud resources. |

[Back to README](../README.md)
