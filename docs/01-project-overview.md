# Project overview

Purpose: This document explains why ShipIt exists, what the student will build, and how the scope stays small enough for six weeks.

## Problem statement

ShortLink is a small URL-shortener service. The startup deploys it with `scp` and manual restarts. A release can break production because there is no repeatable build, no deployment gate, and no rollback practice. The team also lacks metrics, alerts, and runbooks, so slow redirects are found by users first.

ShipIt turns this manual process into a local cloud-native delivery platform. The student builds the ShortLink application and the platform that packages, tests, deploys, observes, and restores it. The platform runs locally with Docker, kind, Helm, Terraform, and Kubernetes.

## Vision

ShipIt MUST let a fresher show the complete DevOps lifecycle for one microservice. A reviewer MUST see a secure container image, a green CI pipeline, two Kubernetes environments, dashboards, alerts, backup and restore scripts, and documented reliability drills.

## Measurable goals

| Goal | Target |
|---|---|
| Safe releases | A rolling update under k6 load has 0 failed requests. |
| Fast redirects | With 50 virtual users and 2 prod replicas, p95 redirect latency is below 300 ms. |
| Secure image | The final image is below 200 MB and has 0 CRITICAL Trivy findings. |
| Quality gates | Every pull request runs lint, tests, coverage, image scan, secret scan, Helm checks, Kubernetes schema checks, and Terraform validation. |
| Local reproducibility | A clean clone starts the lite profile on an 8 GB laptop. |
| Alert response | Each mandatory alert has a runbook and a recorded drill or demo step. |

## Scope by priority

### Must: project MVP

- FastAPI ShortLink API with PostgreSQL, JSON logs, health, readiness, and Prometheus metrics.
- Multi-stage Docker image, Docker Compose development stack, and operational Bash scripts.
- GitHub Actions CI on pull requests and image publishing from `main` to GHCR.
- kind cluster with `shortlink-dev` and `shortlink-prod` namespaces.
- Helm chart named `shortlink` with separate dev and prod values.
- Gateway API routing with Envoy Gateway.
- Terraform 1.16.x that creates kind and installs Envoy Gateway and kube-prometheus-stack.
- Monitoring, Mailpit alert e-mail, runbooks, and reliability drills.
- Security scans, dependency hygiene, and no secrets in Git.
- Local-to-AWS mapping for interview readiness.

### Should: recommended hardening

- Argo CD GitOps with automatic dev sync and manual prod promotion.
- Loki and Grafana Alloy logs.
- HPA, PodDisruptionBudget, multi-node kind, and NetworkPolicy enforcement.
- Deeper tests, SBOM, release workflow, TFLint, yamllint, helm-unittest, and Terraform tests.
- SOPS or Sealed Secrets, SLO dashboard, postmortem, DORA metrics, and Jenkins explanation.

### Could: stretch items

- Ansible for a fresh Ubuntu VM.
- Argo Rollouts canary delivery.
- cosign signing and Kyverno policy.
- OpenTelemetry tracing.
- A short AWS free-tier k3s demo with a budget warning and destroy step.
- CloudNativePG evaluation.

## Non-goals and out of scope

- ShipIt MUST NOT use managed Kubernetes for Must scope.
- ShipIt MUST NOT require a cloud account.
- ShipIt MUST NOT use Bitnami images or charts.
- ShipIt MUST NOT use ingress-nginx.
- ShipIt MUST NOT implement a service mesh or multi-cluster design.
- ShipIt MUST NOT store real personal data or real production URLs in examples.
- ShipIt MUST NOT include implementation code in this specification repository.

## Assumptions

| Assumption | Reason |
|---|---|
| Students know basic Python and SQL. | The project teaches DevOps around a small app. |
| Students use Windows 11 with WSL2, macOS, or Linux. | These platforms can run Docker and kind locally. |
| The trainer validates the lite profile on an 8 GB laptop before week 1. | The Must scope has to work on modest hardware. |
| The student repository is named `shipit`. | The brief fixes this repository name. |
| The specification repository is named `spec-shipit-devops-platform`. | The trainer publishes this spec publicly. |

## Constraints

| Constraint | Exact value |
|---|---|
| Duration | 6 weeks, about 180 hours. |
| Must effort budget | At most about 120 hours, including about 30 hours of guided learning and setup. |
| Remaining time | About 60 hours for Should items, interview work, and contingency. |
| Minimum hardware | 8 GB RAM with the lite profile. |
| Recommended hardware | 16 GB RAM with the standard profile. |
| Kubernetes target | kind 0.33.x with Kubernetes 1.37.x. |
| IaC target | Terraform 1.16.x by default. OpenTofu needs an ADR. |
| Gateway implementation | Envoy Gateway 1.9.x. |
| Helm target | Helm 4.x. |
| Alert receiver | Mailpit e-mail is Must. A webhook receiver is Should. |

The spec uses `.localhost` host names for routes. Local smoke tests use Envoy port-forward to `127.0.0.1:8080` and send the expected Host header. Browser demos MAY add hosts-file entries to `127.0.0.1`.

## Lite and standard profiles

| Profile | Hardware | Enabled services | Limits |
|---|---|---|---|
| Lite | 8 GB RAM | 1-node kind, ShortLink dev and prod, two PostgreSQL StatefulSets, Envoy Gateway, kube-prometheus-stack with short retention, Mailpit | Prometheus retention 6 hours; no Loki; no Argo CD; app memory limit 256 MiB; PostgreSQL memory limit 512 MiB per environment. |
| Standard | 16 GB RAM | Lite services plus optional multi-node kind, Argo CD, Loki, Alloy, metrics-server, and Calico | Prometheus retention 24 hours; app memory limit 512 MiB; PostgreSQL memory limit 1 GiB per environment. |

`bootstrap.sh` MUST detect host RAM. If RAM is below 16 GB, it MUST recommend the lite profile before cluster creation.

## Success criteria

- A reviewer can create a short link at `POST /api/v1/links`.
- A reviewer can open `http://shortlink.localhost:8080/{code}` and receive the exact redirect status from the platform specification.
- Dev and prod host names do not collide: `shortlink-dev.localhost:8080` and `shortlink.localhost:8080` for browser demos.
- Each environment has its own PostgreSQL StatefulSet.
- The pull request checks block a broken change.
- The `main` branch publishes a GHCR image tagged with SemVer and short SHA.
- Grafana shows RED metrics and resource usage.
- Alertmanager sends at-least-once alert e-mail to Mailpit.
- A backup can be restored into the local PostgreSQL database.
- The final demo includes rolling update, rollback, and pod-kill evidence.

## Skills learned and job relevance

| Skill learned | Typical job-description keywords |
|---|---|
| Linux and Bash operations | Shell scripting, process checks, permissions, cron-style automation |
| Docker packaging | Multi-stage build, slim image, non-root container, Compose |
| GitHub Actions CI/CD | PR checks, required status checks, GHCR, release tags |
| Kubernetes operations | Deployments, Services, StatefulSets, probes, resources, Gateway API |
| Helm packaging | Charts, values, release history, rollback, test hooks |
| Terraform | State, providers, modules, variables, outputs, drift |
| Observability | Prometheus, Grafana, RED metrics, Alertmanager, runbooks |
| Security | Trivy, Gitleaks, Dependabot, secrets management, least privilege |
| Reliability | Zero-downtime deploys, rollback, self-healing, backup and restore |
| Cloud literacy | EKS, ECR, EBS, load balancers, SNS, SES, S3 remote state |

## Related documents

- [Users and roles](02-users-and-roles.md) defines actors, stories, and permissions.
- [Functional requirements](03-functional-requirements.md) defines feature behavior and business rules.
- [Non-functional requirements](04-non-functional-requirements.md) defines measurable quality targets.
- [System architecture](05-system-architecture.md) will describe diagrams and ADRs.
- [Tech stack and setup](06-tech-stack-and-setup.md) will pin versions and setup steps.
- [Data model](07-data-model.md) defines ShortLink data and operational records.
- [Platform specification](08-platform-specification.md) will define endpoints, resources, pipelines, and alerts.

[Back to README](../README.md)
