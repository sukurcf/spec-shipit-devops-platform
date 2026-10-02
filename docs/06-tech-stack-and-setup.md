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
| Dashboard | Grafana OSS | 13.x | Show RED metrics and resource panels. | It is widely used in SRE roles. |
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
| TypeScript tooling for dashboards or helper UI | Why is a frontend needed for this DevOps project? | It MUST NOT distract from Must platform work. |
| AWS free-tier k3s demo | Why is cloud cost acceptable for a short demo? | It MUST include a budget warning and a destroy-after-demo note. |

## Not allowed

| Tool or pattern | Reason |
|---|---|
| ingress-nginx | The cohort uses Gateway API with Envoy Gateway because ingress-nginx is retired. |
| Bitnami images or charts | The free catalog changed in 2025, and Must PostgreSQL uses a direct StatefulSet. |
| Docker image tags with `latest` | They break reproducibility and security evidence. |
| Committed `.env` files with secrets | They violate NFR-SEC-03 and fail Gitleaks. |
| Promtail | It is deprecated. Use Grafana Alloy for optional logs. |
| Managed Kubernetes for Must scope | The Must scope must run locally without a cloud account. |
| A PostgreSQL operator for Must scope | The brief requires a StatefulSet, PVC, and Service only. |
| Workflow YAML copied from a tutorial without explanation | Students must explain every gate in viva. |

## Hardware profiles

| Profile | Laptop target | Cluster shape | Enabled services | Exact limits |
|---|---|---|---|---|
| Lite | 8 GB RAM | 1-node kind | ShortLink dev and prod, two PostgreSQL StatefulSets, Envoy Gateway, kube-prometheus-stack, Mailpit | Prometheus retention 6 hours; no Loki; no Argo CD; app memory limit 256 MiB per pod; PostgreSQL memory limit 512 MiB per environment. |
| Standard | 16 GB RAM | 1-node kind for Must; 3-node kind is Should | Lite services plus optional metrics-server, Calico, Loki, Alloy, and Argo CD | Prometheus retention 24 hours; app memory limit 512 MiB per pod; PostgreSQL memory limit 1 GiB per environment. |

The lite profile MUST use a 1-node kind cluster, Prometheus with 6-hour retention, no Loki, and no Argo CD. The lite profile uses `.wslconfig` values `memory=4GB` and `swap=4GB` because both environments and monitoring run together.

| Component | Lite budget | Standard budget |
|---|---:|---:|
| kind node overhead | 700 MiB | 1,200 MiB |
| Envoy Gateway controller and proxy | 192 MiB | 384 MiB |
| Prometheus | 640 MiB | 1,100 MiB |
| Grafana | 192 MiB | 320 MiB |
| Alertmanager | 96 MiB | 160 MiB |
| Prometheus Operator | 192 MiB | 256 MiB |
| kube-state-metrics | 96 MiB | 160 MiB |
| node-exporter | 64 MiB | 96 MiB |
| Mailpit | 64 MiB | 128 MiB |
| ShortLink app pods | 512 MiB total | 1,024 MiB total |
| Two PostgreSQL pods | 768 MiB total | 1,800 MiB total |
| **Steady total** | **3,516 MiB** | **6,628 MiB** |

Trainer pre-check validates these budgets with both environments deployed. Helm install and image-pull peaks MAY use swap briefly, but steady state MUST fit inside WSL memory.

## Windows 11 with WSL2 checklist

| Step | Check |
|---|---|
| 1 | Install WSL2 with Ubuntu 24.04. |
| 2 | Set `.wslconfig` for an 8 GB laptop to `memory=4GB` and `swap=4GB`. |
| 3 | Set `.wslconfig` for a 16 GB laptop to `memory=8GB` and `swap=4GB`. |
| 4 | Install Docker Desktop with the WSL2 backend, or an approved Docker Engine setup in WSL2. |
| 5 | Install Git, uv, kubectl 1.37.x, kind 0.33.x, Helm 4.x, Terraform 1.16.x, k6 2.x, and VS Code Remote WSL. |
| 6 | Confirm Docker can run Linux containers from Ubuntu. |
| 7 | Run the future `bootstrap.sh` check mode before creating the cluster. |
| 8 | If RAM is below 16 GB, select the lite profile. |

## macOS checklist

| Step | Check |
|---|---|
| 1 | Use Apple Silicon or Intel macOS with Docker Desktop or a compatible local Docker engine. |
| 2 | Allocate at least 4 GB memory to Docker for lite, or 8 GB for standard. |
| 3 | Install Git, uv, kubectl 1.37.x, kind 0.33.x, Helm 4.x, Terraform 1.16.x, k6 2.x, and VS Code. |
| 4 | Verify `.localhost` host names resolve to loopback. |
| 5 | Use lite mode if Docker memory is below 8 GB. |

## Linux checklist

| Step | Check |
|---|---|
| 1 | Use Ubuntu 24.04 or a similar modern distribution. |
| 2 | Install Docker Engine 29.x and add the user to the Docker group only if approved by the trainer. |
| 3 | Install Git, uv, kubectl, kind, Helm, Terraform, k6, and VS Code or another editor. |
| 4 | Confirm cgroup v2 and container networking work before week 1 ends. |
| 5 | Use the hosts-file fallback if `shortlink.localhost` does not resolve. |

## Bootstrap sequence

| Order | Action |
|---:|---|
| 1 | Run `bootstrap.sh --check-only` and select `lite` or `standard`. |
| 2 | Create the kind cluster and platform namespaces. |
| 3 | Create the dev Secret with `scripts/create-secrets.sh` from an ignored local file. |
| 4 | Create the prod Secret with `scripts/create-secrets.sh` from an ignored local file. |
| 5 | Install or upgrade the ShortLink Helm releases. |
| 6 | Start Envoy port-forward and run `smoke-test.sh`. |

## Trainer pre-check

- Validate the lite profile on an 8 GB laptop before week 1.
- Deploy both `shortlink-dev` and `shortlink-prod` during the pre-check.
- Confirm `memory=4GB` and `swap=4GB` work for WSL2 lite.
- Create a 1-node kind 0.33.x cluster with Kubernetes 1.37.x.
- Create dev Secrets with `scripts/create-secrets.sh` from an ignored local file.
- Create prod Secrets with `scripts/create-secrets.sh` from an ignored local file.
- Install Envoy Gateway 1.9.x and kube-prometheus-stack 91.x.
- Confirm Prometheus retention is 6 hours and both PostgreSQL pods stay below the lite budget.
- Confirm Loki and Argo CD are absent in lite.
- Confirm `shortlink-dev.localhost` and `shortlink.localhost` resolve or have hosts-file fallback.
- Record any trainer-only machine issue as a clarification issue.

## Free accounts needed

| Account | Required? | Purpose |
|---|---|---|
| GitHub | Must | Student repository, pull requests, GitHub Actions, GHCR, Dependabot. |
| Docker Hub | Should | Pull base images if anonymous rate limits block setup. |
| Grafana Cloud | No | Not needed because Grafana runs locally. |
| AWS | No for Must | Only Could work uses AWS, and it needs a cost warning. |

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
| 8 | Prometheus, Grafana, Alertmanager, RED, USE, SLI, SLO, and SLA | 5 | Student can read dashboards and alerts. |
| 9 | Security scans, secrets, Dependabot, and vulnerability triage | 3 | Student can justify Trivy and Gitleaks failures. |
| 10 | Interview revision and AWS basics | 4 | Student can map local components to AWS services. |

[Back to README](../README.md)
