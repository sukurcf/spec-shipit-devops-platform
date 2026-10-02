# Non-functional requirements

Purpose: This document defines measurable quality targets for ShipIt and names the evidence used to verify each target.

## NFR catalog

| ID | Category | Priority | Target | Verification method |
|---|---|---|---|---|
| NFR-PERF-01 | Redirect performance | Must | With 50 k6 virtual users, 2 prod replicas, redirect following disabled, and warmed cache disabled, valid-redirect p95 is below 300 ms for 5 minutes. | k6 report on the reference 16 GB standard profile. Students with 8 GB run the same script at 20 virtual users and save results. |
| NFR-PERF-02 | API performance | Must | `POST /api/v1/links` p95 is below 500 ms for 5 minutes with 20 virtual users. | k6 API scenario report. |
| NFR-PERF-03 | Image size | Must | The final ShortLink image is smaller than 200 MB. | CI image inspection step. |
| NFR-REL-01 | Rolling update | Must | A rolling update under load has 0 failed requests. | Reliability drill evidence. |
| NFR-REL-02 | Self-healing | Must | Kubernetes replaces a killed app pod within 120 seconds. | Pod-kill drill log and dashboard timestamp. |
| NFR-REL-03 | Backup recovery | Must | `db-restore.sh` restores a chosen backup into the target local database without losing the backup file. | Restore drill with row-count evidence. |
| NFR-SEC-01 | Runtime security | Must | App containers run as non-root, drop ALL capabilities, use a read-only root filesystem, and define resource limits. | Rendered manifest review and runtime check. |
| NFR-SEC-02 | Vulnerability gate | Must | Trivy reports 0 CRITICAL image vulnerabilities and no HIGH vulnerability with a fix available. | CI Trivy report. |
| NFR-SEC-03 | Secret hygiene | Must | Gitleaks finds 0 committed secrets, and Kubernetes Secrets are created from a local ignored file. | CI secret scan and `.gitignore` review. |
| NFR-SEC-04 | Dependency hygiene | Must | Dependabot covers GitHub Actions, pip, and Docker dependencies. | Dependabot configuration review. |
| NFR-OBS-01 | Metrics coverage | Must | Prometheus scrapes app request count, latency histogram, redirect count, CPU, memory, and pod restarts. | ServiceMonitor and Grafana dashboard review. |
| NFR-OBS-02 | Alert coverage | Must | High 5xx rate, high p95 latency, crash-looping pod, and target-down alerts each have a runbook link. | PrometheusRule and runbook review. |
| NFR-MAINT-01 | Python quality | Must | Ruff passes, and app tests meet line coverage at least 80% and branch coverage at least 70%. | Pull request CI report. |
| NFR-MAINT-02 | Shell quality | Must | All required scripts use strict mode, show help text, and pass ShellCheck with 0 findings. | CI ShellCheck report. |
| NFR-PORT-01 | Operating systems | Must | Must scope works on Windows 11 WSL2 Ubuntu, macOS, and Linux. | Setup notes and at least one clean-clone demo. |
| NFR-HW-01 | Lite profile | Must | The lite profile runs on 8 GB RAM with 1-node kind, no Loki, no Argo CD, and Prometheus retention of 6 hours. | Trainer pre-check before week 1. |
| NFR-HW-02 | Bootstrap guidance | Must | `bootstrap.sh` warns below 16 GB RAM and recommends the lite profile. | Script test and screenshot. |
| NFR-CI-01 | CI enforcement | Must | A pull request fails when a mandatory gate fails. | Negative CI evidence for one intentionally broken branch. |
| NFR-DOC-01 | Documentation | Must | The student README starts the Must system from a clean clone in 10 steps or fewer. | Trainer clean-clone review. |
| NFR-LIC-01 | Licence compliance | Must | All Must tools are free for local training use; Redis is optional and Valkey is the preferred permissive cache. | Licence notes and ADR where needed. |
| NFR-PRIV-01 | Privacy awareness | Must | Examples use fictional data, and docs warn that URLs can contain personal data under DPDP Act 2023 awareness. | Data review. |
| NFR-SCALE-01 | Horizontal scale | Should | HPA keeps app replicas between 2 and 6 at 70% CPU target. | HPA load test evidence. |
| NFR-NET-01 | Network isolation | Should | Only ShortLink pods can connect to PostgreSQL when NetworkPolicy is enabled. | Positive app test and negative debug-pod test. |
| NFR-LOG-01 | Logs | Should | Loki and Alloy retain local logs for 24 hours in the standard profile. | Grafana Explore evidence. |
| NFR-SLO-01 | SLO visibility | Should | Dashboard shows 99.5% availability and p95 latency below 300 ms as design targets. | SLO dashboard export. |

## Performance and scalability notes

- The reference performance test uses the standard 16 GB profile at 50 virtual users.
- Students with 8 GB laptops run the same k6 script at 20 virtual users and record results.
- The trainer MAY rerun the 50-virtual-user script on a 16 GB machine during the final demo.
- Redirect tests MUST disable redirect following.
- Redirect tests MUST measure valid 302 redirects separately from expected 404 misses.
- Create-link tests MUST use unique target URLs under `https://example.in/`.
- HPA is Should because a 1-node lite cluster cannot show useful scaling.

## Reliability requirements

| Scenario | Required behavior |
|---|---|
| App pod killed | The Deployment creates a replacement pod within 120 seconds. |
| Bad release deployed | `helm rollback` restores the previous working release. |
| Rolling update under k6 load | No request fails during the update. |
| PostgreSQL backup restore | The restore target contains the restored link rows and the current backup is retained. |
| Target down alert | Mailpit receives at least 1 alert e-mail with a runbook link. |

## Security requirements

ShipIt MUST address practical OWASP Top 10 risks for this service.

| OWASP area | ShipIt requirement | Evidence |
|---|---|---|
| Broken access control | Optional delete endpoint MUST require `X-API-Key`. Missing or wrong keys return 401. | API security tests. |
| Cryptographic failures | Secrets MUST stay out of Git and image layers. | Gitleaks and image-layer review. |
| Injection | Database access MUST use parameterized driver or ORM calls. | Code review and integration tests. |
| Security misconfiguration | Pods MUST run with least privilege and read-only root filesystem. | Manifest validation. |
| Vulnerable components | Trivy and Dependabot MUST run in CI. | CI reports. |
| Logging and monitoring failures | JSON logs and Prometheus metrics MUST include request outcomes. | Logs and dashboard review. |

## Privacy and DPDP Act awareness

ShortLink stores URLs. A URL can contain personal data in the path or query string. Students MUST use only fictional examples such as `https://example.in/careers`. The project is a training system and is not a legal compliance product.

## Hardware and portability requirements

| Item | Lite profile | Standard profile |
|---|---|---|
| RAM | 8 GB | 16 GB |
| kind nodes | 1 node | 1 node Must; 3 nodes Should |
| Prometheus retention | 6 hours | 24 hours |
| Loki and Alloy | Off | Should |
| Argo CD | Off | Should |
| App limit | 256 MiB per pod | 512 MiB per pod |
| PostgreSQL limit | 512 MiB per environment | 1 GiB per environment |

Per-component memory budget for trainer pre-check:

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

Windows WSL2 guidance MUST be consistent with document 06. For 8 GB laptops, use WSL memory of 4 GB and swap of 4 GB. For 16 GB laptops, use WSL memory of 8 GB and swap of 4 GB.

## Maintainability requirements

- The application MUST use Python 3.12.x.
- The project MUST commit `uv.lock`.
- Ruff MUST lint and format application code.
- mypy MUST check application packages with typed functions.
- Tests MUST exclude migrations, generated files, and `__main__` blocks from coverage only when documented.
- Scripts MUST contain usage help and clear failure messages.
- ADRs MUST record choices for Terraform versus OpenTofu, cache technology if implemented, Gateway implementation, log stack, and secret encryption if implemented.

## Licence compliance

- Terraform 1.16.x is the default IaC tool because it is the job-market keyword for this cohort.
- OpenTofu 1.13.x MAY be used only with an ADR.
- Valkey is the preferred optional cache when a permissive licence is needed.
- Bitnami images and charts MUST NOT be used.
- Tool versions MUST follow the cohort reference table or a checked official source.

[Back to README](../README.md)
