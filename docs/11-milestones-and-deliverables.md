# Milestones and deliverables

Purpose: This document gives the ShipIt effort budget, six-week plan, deliverables, submission process, and final demo script.

## Effort budget

The total project budget is about 180 hours. Must work is about 120 hours, including about 30 hours of learning and setup.

| Feature area | Linked IDs | Must hours | Should or contingency hours | Notes |
|---|---|---:|---:|---|
| Guided learning and local setup | NFR-PORT-01, NFR-HW-01 | 30 | 0 | Counts inside the Must budget. Extra learning is optional and outside Must. |
| ShortLink app and database | FR-APP-01 to FR-APP-04 | 16 | 6 | Should time covers alias, expiry, stats, delete, and cache. |
| Containers, Compose, and scripts | FR-CON-01, FR-CON-02, FR-BASH-01 | 11 | 4 | Includes backup and restore scripts. |
| GitHub Actions and security gates | FR-CI-01 to FR-CI-03, FR-SEC-01, FR-SEC-02 | 12 | 5 | Should time covers release workflow, SBOM, and extra lint. |
| Kubernetes and Gateway | FR-K8S-01 to FR-K8S-06 | 14 | 8 | Should time covers HPA, PDB, NetworkPolicy, and multi-node kind. |
| Helm and Terraform | FR-HELM-01, FR-HELM-02, FR-IAC-01, FR-IAC-02 | 11 | 6 | Should time covers helm-unittest, TFLint, and Terraform tests. |
| Observability, alerting, and runbooks | FR-OBS-01 to FR-OBS-04, FR-REL-01 | 10 | 8 | Should time covers Loki, Alloy, SLO dashboard, and burn alerts. |
| Reliability drills and evidence | FR-REL-02 | 6 | 5 | Must evidence is complete by the end of week 4. |
| Documentation, interview, and final polish | FR-DOC-01 | 10 | 18 | AWS mapping and README setup are Must; extra polish is week 6. |
| **Total** |  | **120** | **60** | Could items start only after all Must items pass. |

## Week-by-week plan

| Week | Goals | Tasks that name FR IDs | Deliverables | Friday demo checkpoint |
|---|---|---|---|---|
| 1 | Prepare the laptop and prove a thin slice. | Set up WSL2 or local tools; create `shipit`; add GitHub Flow; start FR-APP-03 health; draft ADRs for FR-IAC-01 and FR-K8S-02; sketch FR-DOC-01 mapping. | Repository skeleton, README setup draft, first CI skeleton, health endpoint, local PostgreSQL decision notes. | Show `bootstrap.sh` check output, a passing health request, and the first PR with CI status. |
| 2 | Build ShortLink core and local containers. | Complete FR-APP-01, FR-APP-02, FR-APP-04, FR-CON-01, FR-CON-02; start FR-BASH-01 backup and smoke scripts. | Create and redirect endpoints, tests, Docker image below 200 MB, Compose stack, JSON logs. | Create `https://example.in/careers`, redirect `Ab3kP91`, show logs and image-size evidence. |
| 3 | Add CI gates and Kubernetes dev. | Complete FR-CI-01, FR-CI-02, FR-CI-03, FR-K8S-01, FR-K8S-02, FR-K8S-03, FR-K8S-04 for dev; add FR-SEC-01 scans and immutable GHCR tag checks. | Required PR checks, branch protection screenshot, GHCR SemVer and short-SHA publication evidence, dev namespace, Gateway route, ConfigMap, Secret workflow. | Break coverage intentionally in a branch, show merge blocked, then show dev health through `shortlink-dev.localhost:8080`. |
| 4 | Finish prod platform and Must observability. | Complete FR-K8S-05, FR-K8S-06, FR-HELM-01, FR-HELM-02, FR-IAC-01, FR-IAC-02, FR-OBS-01 to FR-OBS-04, FR-REL-01, FR-REL-02, FR-SEC-02, and FR-DOC-01. | Helm chart, dev and prod releases, Terraform apply evidence, dashboards, alerts, runbooks, reliability drill evidence, AWS mapping, Dependabot. | Show prod 2 replicas, Grafana RED panels, a Mailpit alert, reliability evidence, AWS mapping, and Terraform outputs. |
| 5 | Add selected Should items and harden evidence. | Choose from FR-GITOPS-01, FR-LOG-01, FR-SCALE-01, FR-K8S-07, FR-NET-01, FR-TEST-01, FR-JENKINS-01. | Rolling update evidence, rollback evidence, pod kill evidence, optional hardening PRs. | Run k6 during a prod update and show 0 failed requests. Explain GitHub Actions versus Jenkins. |
| 6 | Final polish, demo, and interview readiness. | Close test gaps, update ADRs, polish README, prepare viva for all Must IDs. | Final repository, coverage report, ADRs, runbooks, screenshots or logs, 5-minute demo video, submission issue. | Run the full 10-minute final demo script and answer viva questions. |

## Gantt chart

```mermaid
gantt
  title ShipIt six-week delivery plan
  dateFormat  YYYY-MM-DD
  axisFormat  %d %b
  section Setup and learning
  Laptop setup and DevOps basics :a1, 2026-10-05, 5d
  Repository and first health slice by Friday :a2, 2026-10-06, 3d
  section Application and containers
  ShortLink API and PostgreSQL :b1, 2026-10-12, 7d
  Docker, Compose, and scripts :b2, 2026-10-15, 5d
  section Platform gates
  CI and security gates :c1, 2026-10-19, 6d
  Kubernetes dev and Gateway :c2, 2026-10-21, 5d
  section Release platform
  Helm and Terraform :d1, 2026-10-26, 5d
  Observability and alerts :d2, 2026-10-28, 5d
  section Hardening and proof
  Should hardening and reliability replay :e1, 2026-11-02, 4d
  Selected Should items :e2, 2026-11-04, 5d
  section Final readiness
  Documentation polish and interview mapping :f1, 2026-11-09, 3d
  Demo video and viva practice :f2, 2026-11-12, 2d
```

## Final deliverables checklist

| Deliverable | Required evidence |
|---|---|
| Public `shipit` repository | Trainer `@sukurcf` is a collaborator. |
| Student README | Clean clone starts Must system in 10 steps or fewer. |
| Architecture diagram | Shows app, PostgreSQL, kind, Envoy Gateway, Prometheus, Grafana, Alertmanager, and Mailpit. |
| ADRs | At least 5 ADRs covering Terraform/OpenTofu, Gateway, PostgreSQL design, secret handling, and optional cache or logs. |
| Test and coverage report | Shows line coverage at least 80% and branch coverage at least 70%. |
| CI evidence | Required checks on `main` are green. |
| Security evidence | Gitleaks clean, Trivy clean by policy, no secrets in history. |
| Helm evidence | Dev and prod values render; prod has 2 replicas; `helm test` passes. |
| Terraform evidence | Apply output shows cluster name, dev host, prod host, and Grafana local URL. |
| Monitoring evidence | Grafana RED dashboard has rate, errors, p50, p95, p99, CPU, memory, and restarts. |
| Alert evidence | Mailpit receives one alert e-mail with runbook link. |
| Reliability evidence | Rolling update has 0 failed requests; rollback names revision; pod kill recovers within 120 seconds. |
| Demo video | Maximum 5 minutes, focused on working system evidence. |

## Submission process

1. Freeze feature work after all Must gates pass.
2. Open a final submission issue in the specification repository with the `shipit` repository link.
3. Add links to the final PR list, CI run, coverage report, demo video, runbooks, and ADRs.
4. Confirm that no real secret or personal data is in Git history.
5. Confirm AWS Could work is destroyed if it was attempted.
6. Ask the trainer to schedule the final viva.

## Ten-minute demo script

| Minute | What to show | Evidence to mention |
|---:|---|---|
| 0 | State the problem and architecture in one sentence. | Manual `scp` changed into a local delivery platform. |
| 1 | Open the GitHub repository and latest green PR. | Branch protection and required checks. |
| 2 | Create a ShortLink for `https://example.in/careers`. | 201 response with 7-character code and JSON log. |
| 3 | Open `http://shortlink.localhost:8080/{code}`. | 302 Found with `Location` target. |
| 4 | Show Docker image evidence. | Image below 200 MB, non-root, no CRITICAL Trivy findings. |
| 5 | Show kind environments. | `shortlink-dev`, `shortlink-prod`, prod has 2 app pods. |
| 6 | Show Helm and Terraform evidence. | Chart version, release revision, and Terraform outputs. |
| 7 | Show Grafana dashboard and a Mailpit alert. | RED metrics, resource panels, and runbook link. |
| 8 | Run or replay rolling update drill. | k6 has 0 failed requests. |
| 9 | Explain AWS mapping and answer one trade-off. | kind to EKS, GHCR to ECR, PVC to EBS, Gateway to load balancer, Alertmanager e-mail to SNS or SES, local state to S3. |
| 10 | End with next improvements. | One Should item and one Could item with risks. |

[Back to README](../README.md)
