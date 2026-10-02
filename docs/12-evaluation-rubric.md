# Evaluation rubric

Purpose: This document defines ShipIt mandatory gates, weighted scoring, bonus conditions, deductions, grade bands, and viva questions.

## Mandatory gates

If any gate fails, the result is Rework required for any score.

| Gate | ShipIt evidence |
|---|---|
| CI is green on `main` | Latest `main` run has all required checks from BR-15. |
| Coverage thresholds are met | App tests show line coverage at least 80% and branch coverage at least 70%. |
| Clean clone starts/stops both local stages | After download/cache/secret preparation, README/document 06 starts Compose first, then one-node kind with loaded local images. TC-LOCAL-001 to TC-LOCAL-004 pass for both stages without external egress, proving exact seed demo, health, persistence, confirmed reset, and clear input/dependency failures. |
| No secrets in Git history | Gitleaks is clean and the secret source file is ignored. |
| Student can explain code in viva | Student explains one app flow, one CI gate, one Helm value, one Terraform output, and one alert. |

## Weighted rubric

| Category | Points |
|---|---:|
| Functional completeness (Must FRs work in the demo) | 30 |
| Code quality and design | 15 |
| Testing and coverage | 15 |
| DevOps: CI/CD, Docker, reproducibility | 10 |
| Documentation (README, ADRs, API/CLI/pipeline docs) | 10 |
| Git and engineering practices (PRs, commits, issues) | 5 |
| Final demo and viva | 15 |
| **Total** | **100** |

Version 1.1 removes frontend/custom visualization and cloud-deployment assessment without changing weights. Their effort/points now assess Python/API contracts, Prometheus/alert JSON, local artifact/image handling, and reproducible operations/testing/documentation. Built-in Grafana is optional operations tooling only, never a required interface or bonus. The former cloud bonus path is replaced by FR-LOCAL-02 local inventory validation.

## Project-specific criteria

| Category | Excellent | Good | Needs work |
|---|---|---|---|
| Functional completeness | All Must FRs including FR-LOCAL-01 work; Compose then kind, sequential dev/prod isolation, profile replicas, API/metric/alert/runbook/backup/restore/drill evidence. | Core backend/local DevOps and all acceptance gates work; a minor evidence item needs cleanup. | A Must capability is missing: local persistence/errors, profile replicas (lite prod 1/standard 2), Mailpit alerts, Terraform, or rollback. |
| Code quality and design | FastAPI layers are clear; config is 12-factor; database access is parameterized; Bash scripts are safe; Kubernetes resources use probes and hardening consistently. | Design is understandable and safe, but one area has duplication or weak names that do not block operation. | App, scripts, or manifests are difficult to explain, mix responsibilities, or skip required hardening. |
| Testing and coverage | At least 45 meaningful cases; unit/API/script/security/platform/performance tests and all four local cases per stage cover core/negative paths; 80% line/70% branch gates. | Coverage/local gates pass; some additional boundary evidence could improve. | Coverage lacks meaningful assertions, local cases fail, or unavailable-dependency paths are untested. |
| DevOps: CI/CD, Docker, reproducibility | PR gates catch broken code, image issues, secrets, Helm errors, schema errors, and Terraform errors; clean clone works on lite. | CI and local setup work, but one optional hardening check remains manual. | Required gates are missing, image is too large, `latest` tags are used, or local setup needs trainer repair. |
| Documentation | README/ADRs/runbooks/API evidence and interview theory match the implementation; start/stop/demo/reset, caches/secrets, loopback ports, persistence, failures, proposed versus observed caps, and WSL 4GB/4GB (8GB host) or 8GB/4GB (16GB host) are explicit. | Documentation is correct but misses a useful command interpretation. | README cannot run locally, hidden credentials/registry/cloud dependency exists, ADRs are shallow, or runbooks lack query/CLI diagnosis. |
| Git and engineering practices | PRs are small, linked to IDs, use Conventional Commits, include checklists, and explain AI help when used. | Most work uses PRs and good commits, with minor naming or issue-link gaps. | Direct pushes, huge PRs, unclear commits, or missing PR descriptions make review hard. |
| Final demo and viva | The 10-minute demo is smooth; student explains Linux, Docker, Kubernetes, CI/CD, Terraform, Helm, monitoring, and AWS mapping with ShipIt examples. | Demo covers all Must evidence, but answers need a few trainer prompts. | Student cannot explain code, pipeline gates, cluster resources, state, alerts, or rollback. |

## Bonus rules

Bonus is up to +10 points. Bonus applies only to Could items. Bonus applies only when the base score is 60 or more and all mandatory gates pass. The final score is capped at 100.

| Bonus item | Maximum points | Conditions |
|---|---:|---|
| Ubuntu VM automation | 2 | FR-ANSIBLE-01 prepares a fresh Multipass Ubuntu VM with Docker and kind. |
| Canary delivery | 2 | FR-ROLLOUT-01 uses Argo Rollouts and proves a 10% canary before promotion. |
| Image signing policy | 2 | FR-SIGN-01 signs images with cosign and rejects an unsigned image with Kyverno. |
| Tracing or local artifact validation | 2 | FR-OTEL-01 records an API-to-database trace, or FR-LOCAL-02/TC-INFRA-025 verifies offline artifact/rollback digests and detects drift. No cloud or custom UI path. |
| Database operator evaluation | 2 | FR-DBOP-01 documents CloudNativePG trade-offs after all Must items pass. |

## Deductions

| Issue | Deduction |
|---|---:|
| Real secret committed and then removed without trainer notice | Rework required until history is cleaned. |
| Uses ingress-nginx, Bitnami PostgreSQL chart, or Promtail in Must path | Rework required for that scope. |
| Image uses root or is 200 MB or larger at final demo | Up to 8 points, and gate may fail if CI allows it. |
| Terraform state is committed | Up to 8 points and secret review required. |
| Reliability drill evidence is claimed but not reproducible | Up to 10 points. |
| README steps exceed 10 steps or depend on hidden local files | Up to 6 points. |
| AI-generated solution cannot be explained in viva | Rework required. |
| Submitted custom frontend/visualization or cloud deployment work | No points; remove from assessed scope and complete replacement local/backend evidence. |

## Grade bands

| Score | Grade |
|---:|---|
| 85–100 | Distinction |
| 70–84 | Merit |
| 60–69 | Pass |
| Below 60 | Rework required |

## Ten project-specific viva questions

1. Why does ShipIt use 302 Found for redirects instead of 301? Key points: expiring links; deleted links; browser caching risk.
2. What happens when `/readyz` fails but `/healthz` passes? Key points: pod process alive; database unavailable; Service endpoints remove the pod.
3. How do you prove the Docker image contains no local `.env` secret? Key points: `.dockerignore`; image history; Gitleaks and Trivy evidence.
4. Why does local prod use lite 1/standard 2 replicas with `maxUnavailable: 0` and `maxSurge: 1`? Key points: reserve an extra pod's RAM; loaded local tags; readiness protects old pods; measured k6 rollout.
5. What does Terraform state store in this project? Key points: kind cluster and add-on metadata; local state ignored; AWS remote state maps to S3.
6. How does Envoy Gateway replace older ingress examples? Key points: Gateway API; HTTPRoute; maintained implementation; ingress-nginx not allowed.
7. Explain a Trivy HIGH finding with a fix available. Key points: CI fails; upgrade base or dependency; justify only when no safe fix exists.
8. How does Alertmanager reach Mailpit? Key points: PrometheusRule fires; Alertmanager route; e-mail receiver; runbook link in message.
9. What is the difference between Helm chart values and Kubernetes Secrets? Key points: non-secret configuration; sensitive runtime values; secret source file is ignored.
10. Compare GitHub Actions versus Jenkins for ShipIt. Key points: GitHub-native PR checks; Jenkins controller and agents; same stages can be mapped; Jenkins needs more maintenance.

[Back to README](../README.md)
