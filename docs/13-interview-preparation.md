# Interview preparation

Purpose: This document helps the ShipIt student explain the project, answer fresher interviews, and connect local DevOps tools to AWS basics.

## Explain your project in 2 minutes

Use STAR: Situation, Task, Action, Result.

| Part | Speaking notes |
|---|---|
| Situation | A startup deployed the ShortLink URL shortener by `scp` and manual restarts. Releases were risky, and slow redirects were noticed by users first. |
| Task | I built ShipIt, a local DevOps delivery platform for ShortLink, using Docker, GitHub Actions, kind, Helm, Terraform, Envoy Gateway, Prometheus, Grafana, Alertmanager, and Mailpit. |
| Action | I containerized the app, enforced PR gates, deployed dev and prod namespaces, added probes and hardened pods, created dashboards and alerts, and documented rollback and self-healing drills. |
| Result | The final demo proves green CI, a secure image below 200 MB, prod with 2 replicas, 0 failed requests during rolling update, alert e-mail with runbook, and a clear mapping from local tools to AWS. |

## Interview questions by topic

### Linux

1. What is the difference between a process and a service? Key points: process is running program; service is managed by systemd; logs and restart policy matter.
2. How do you check which process listens on a port? Key points: use `ss` or `lsof`; match port to PID; confirm container port mapping.
3. What do Linux permissions such as `755` mean? Key points: owner, group, others; read, write, execute bits; scripts need execute permission.
4. Why should Bash scripts use strict mode? Key points: stop on failure; catch unset variables; fail pipelines correctly.
5. How would you debug host routing for `shortlink.localhost`? Key points: Envoy port-forward; Host header; hosts-file fallback for browser demos; compare dev and prod host names.

### Docker

6. What is the difference between an image and a container? Key points: image is immutable template; container is running instance; volumes hold persistent data.
7. Why use a multi-stage Docker build? Key points: keep build tools out of runtime; reduce image size; lower attack surface.
8. How do Docker layers affect secrets? Key points: copied secrets can remain in history; `.dockerignore` matters; never pass real secrets at build time.
9. How does Docker Compose help ShipIt? Key points: starts app and PostgreSQL; health checks; local development mirrors dependencies.
10. What is the difference between a Docker network and a Kubernetes Service? Key points: Compose network connects local containers; Service gives stable cluster endpoint; DNS names differ.

### Kubernetes

11. What is a Pod? Key points: smallest deployable unit; one or more containers; shared network namespace.
12. Why use a Deployment for the app and a StatefulSet for PostgreSQL? Key points: app is stateless; database needs stable identity and PVC; separate per environment.
13. What are liveness, readiness, and startup probes? Key points: liveness restarts dead app; readiness controls traffic; startup protects slow boot.
14. Why does ShipIt use Gateway API and HTTPRoute? Key points: modern routing model; Envoy Gateway implementation; ingress-nginx is not used.
15. How do requests and limits help reliability? Key points: scheduler needs requests; limits prevent noisy neighbor; dashboards show CPU and memory.
16. What does `maxUnavailable: 0` protect during prod rollout? Key points: old pods remain available; readiness gates new pods; k6 proves 0 failures.

### CI/CD and Git

17. What is GitHub Flow? Key points: branch from `main`; open PR; checks and review; merge after green CI.
18. Why use Conventional Commits? Key points: readable history; changelog support; scope shows app, Helm, Terraform, or docs.
19. What should happen when coverage falls to 79% line coverage? Key points: CI fails; branch protection blocks merge; add tests or reduce untested code.
20. Why scan both image and manifests? Key points: image vulnerabilities; Kubernetes misconfiguration; both can break security.
21. GitHub Actions versus Jenkins: how are they different? Key points: Actions is GitHub-native; Jenkins needs controller and agents; same lint, test, build, scan stages can be mapped; Jenkins needs more maintenance.

### Terraform and Helm

22. What is Terraform state? Key points: records managed resources; local state is ignored; remote AWS mapping uses S3 and locking.
23. What is the difference between `plan` and `apply`? Key points: plan previews changes; apply changes resources; review plan before apply.
24. What is Terraform drift? Key points: real resource differs from state; plan reveals drift; avoid manual changes.
25. Why use a Terraform module in ShipIt? Key points: reuse cluster or add-on setup; clear inputs; outputs expose endpoints.
26. What is a Helm chart? Key points: package of Kubernetes templates; values customize dev and prod; release history supports rollback.
27. How does `helm rollback` help on-call response? Key points: selects previous revision; restores known working manifests; evidence records revision.

### Monitoring and reliability

28. What are RED metrics? Key points: rate, errors, duration; match HTTP service behavior; dashboard uses p50, p95, p99.
29. What are USE metrics? Key points: utilization, saturation, errors; useful for CPU, memory, disk, and network resources.
30. What is the difference between SLI, SLO, and SLA? Key points: SLI is measurement; SLO is target; SLA is external promise with consequences.
31. Why alert on p95 latency above 500 ms for 10 minutes? Key points: catches sustained user pain; avoids one-off noise; runbook guides diagnosis.
32. How do you prove self-healing? Key points: kill one app pod; Deployment creates replacement; target is within 120 seconds.

### AWS basics

33. What is IAM? Key points: users, roles, policies; least privilege; CI should use short-lived credentials in real cloud projects.
34. What is a VPC? Key points: isolated network boundary; contains subnets; controls routing and security.
35. What is a subnet? Key points: range inside a VPC; public or private; EKS nodes use subnets.
36. What is a security group? Key points: stateful virtual firewall; controls inbound and outbound; maps to allowing Gateway or database traffic.
37. What is EC2? Key points: virtual machine; can run k3s for Could demo; must destroy to avoid cost.
38. What is S3? Key points: object storage; stores Terraform remote state; needs locking through a supported state design.

## Project deep-dive questions

1. How did you prevent broken releases from reaching `main`? Key points: branch protection; required CI checks; no direct push.
2. How did you keep dev and prod separate? Key points: two namespaces; two PostgreSQL StatefulSets; different host names and values.
3. Why is PostgreSQL not deployed through a Bitnami chart? Key points: brief forbids Bitnami; Must uses own StatefulSet; reduces hidden operator behavior.
4. How does the backup script fit the reliability story? Key points: timestamped backup; 7-day rotation; restore drill evidence.
5. How did you decide between lite and standard profiles? Key points: host RAM; 1-node kind in lite; no Loki or Argo CD; Prometheus 6-hour retention.
6. How do you know Alertmanager sent the right alert? Key points: PrometheusRule threshold; Alertmanager route; Mailpit message fields; at-least-once delivery; runbook link.
7. What would you change before running this in production? Key points: managed database; EKS or another managed control plane; remote state; stronger secrets; real incident process.
8. Why is the optional cache not part of Must scope? Key points: PostgreSQL is enough for base project; licence decision needs ADR; cache adds invalidation risks.
9. How did you prove Kubernetes manifests were valid before apply? Key points: Helm render; kubeconform; Gateway API schemas.
10. How do you explain ShipIt to a non-technical manager? Key points: safer releases; faster detection; rollback practice; dashboard evidence.

## Fundamentals check

### Python core

- Explain functions, classes, exceptions, context managers, and dependency injection with the FastAPI app.
- Explain why URL validation belongs before database write.
- Explain why logs must not include `DATABASE_URL` or `API_KEY`.

### SQL

- Explain joins with Link and Redirect event.
- Explain `GROUP BY` for click counts by code.
- Explain a window function for recent latency or redirect ranking.
- Explain transactions when create-link writes a Link row.
- Explain indexes on `code`, `created_at`, and redirect time.

### Git

- Explain branch, commit, rebase, merge, tag, pull request, and branch protection.
- Explain why a force push to `main` is blocked.
- Explain how Dependabot PRs use the same checks.

### Linux

- Know `ps`, `top`, `ss`, `curl`, `dig`, `chmod`, `chown`, `journalctl`, and file paths.
- Explain process exit codes and why scripts fail fast.
- Explain environment variables and why secrets must not be printed.

### HTTP and networking

- Explain DNS, TCP, port, HTTP method, status code, header, and body.
- Explain 201 for create, 302 for redirect, 404 for missing code, 410 for expired or deleted code, and 503 for readiness failure.
- Explain why Gateway routes `shortlink.localhost` to the prod Service.

### How local tools map to AWS services

| Local ShipIt tool | AWS service or concept | Interview explanation |
|---|---|---|
| kind | EKS | Local Kubernetes cluster maps to managed Kubernetes control plane and worker nodes. |
| GHCR | ECR | Container registry stores versioned images for deployments. |
| PVC | EBS | Persistent volume maps to block storage for stateful workloads. |
| Gateway | AWS load balancer | Local Gateway route maps to cloud load balancing in front of services. |
| Alertmanager e-mail | SNS or SES | Local e-mail alerts map to cloud notification or e-mail delivery. |
| Local Terraform state | Remote state in S3 | Shared state should be durable, access-controlled, and locked in a team. |

## Resume bullet templates

- Built ShipIt, a local DevOps platform for a FastAPI URL shortener, with Docker, kind, Helm, Terraform, and Envoy Gateway; achieved `<0>` failed requests during rolling update under `<50>` k6 virtual users.
- Designed GitHub Actions quality gates for Ruff, mypy, pytest coverage, Trivy, Gitleaks, Helm, kubeconform, and Terraform; maintained `<80%+>` line coverage and `<70%+>` branch coverage.
- Implemented Prometheus and Grafana dashboards with RED metrics and resource panels; added Alertmanager to Mailpit with `<4>` alert runbooks.
- Reduced deployment risk by using prod `<2>` replicas, readiness probes, `maxUnavailable` `<0>`, Helm rollback evidence, and pod self-healing within `<120>` seconds.
- Documented cloud mapping from kind, GHCR, PVC, Gateway, Alertmanager, and Terraform state to EKS, ECR, EBS, load balancers, SNS or SES, and S3.

## GitHub and LinkedIn tips

| Area | Tip |
|---|---|
| GitHub profile | Pin `shipit` and write a short description with Docker, Kubernetes, Helm, Terraform, and monitoring keywords. |
| Repository README | Put setup, architecture, screenshots or text evidence, test results, and demo video link near the top. |
| Pull requests | Keep merged PRs visible because interviewers may inspect review habits. |
| LinkedIn headline | Use “Fresher DevOps Engineer | Docker | Kubernetes | Terraform | GitHub Actions”. |
| LinkedIn post | Share one honest post about the rolling update drill and what failed before it worked. |

## Mock-interview checklist

- Can explain the two-minute STAR pitch without reading.
- Can draw the architecture from memory.
- Can explain one failed CI gate and the fix.
- Can compare GitHub Actions versus Jenkins.
- Can explain Docker layer risk for secrets.
- Can explain readiness versus liveness with `/readyz` and `/healthz`.
- Can explain Terraform state, plan, apply, modules, and drift.
- Can explain RED, USE, SLI, SLO, and SLA.
- Can explain IAM, VPC, subnets, security groups, EC2, and S3.
- Can answer what to improve before a real production launch.

[Back to README](../README.md)
