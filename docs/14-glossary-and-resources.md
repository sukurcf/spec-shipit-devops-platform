# Glossary and resources

Purpose: This document defines ShipIt terms and links official documentation for the tools used in the specification.

## Glossary

| Term | Definition |
|---|---|
| ShipIt | The student-built DevOps platform for building, deploying, observing, and recovering ShortLink. |
| ShortLink | The FastAPI URL-shortener microservice deployed by ShipIt. |
| Redirect | An HTTP response that sends the browser from a short code to the target URL. |
| Short code | The public code in `GET /{code}`. Generated codes are 7 Base62 characters. |
| Base62 | Alphabet with digits, uppercase letters, and lowercase letters. It keeps codes readable. |
| Collision | A generated code already exists. ShipIt stops after maximum 5 total generation attempts. |
| 302 Found | Temporary redirect status used so browsers do not permanently cache expiring links. |
| Target URL | The destination URL stored for a short code. It must use `http` or `https`. |
| Health check | A check that confirms the app process can respond on `/healthz`. |
| Readiness check | A check that confirms the app can serve traffic, including PostgreSQL connectivity. |
| Prometheus metric | A numeric time-series value scraped by Prometheus from `/metrics`. |
| RED metrics | Rate, errors, and duration for request-based services. |
| USE metrics | Utilization, saturation, and errors for infrastructure resources. |
| JSON logs | Logs written as structured JSON objects with fields such as request ID and status. |
| Docker image | An immutable package used to create containers. |
| Container | A running instance of an image with isolated process, filesystem, and network view. |
| Layer | A filesystem change in an image. Secrets copied into layers can remain visible. |
| Multi-stage build | A Docker build pattern that keeps build tools out of the final image. |
| Non-root user | A container user without root privileges. It reduces runtime risk. |
| Docker Compose | A local tool that starts multiple containers, such as app and PostgreSQL. |
| kind | Kubernetes in Docker. ShipIt uses it for local clusters. |
| Kubernetes namespace | A logical cluster partition. ShipIt uses `shortlink-dev` and `shortlink-prod`. |
| Deployment | Kubernetes controller for stateless app replicas and rolling updates. |
| StatefulSet | Kubernetes controller for stateful pods such as PostgreSQL. |
| PVC | PersistentVolumeClaim. It requests persistent storage for PostgreSQL. |
| Service | Stable Kubernetes network endpoint for pods. |
| Gateway API | Kubernetes routing API used by ShipIt instead of ingress-nginx. |
| HTTPRoute | Gateway API resource that maps host names and paths to Services. |
| Envoy Gateway | Maintained Gateway API implementation selected for ShipIt. |
| Helm chart | Package of Kubernetes templates, values, and metadata. |
| Values file | Helm configuration file for dev or prod settings. |
| Helm release | Installed chart instance with revision history. |
| Terraform state | File that records resources managed by Terraform. It is local and ignored in Must scope. |
| Terraform module | Reusable Terraform unit with inputs and outputs. |
| kube-prometheus-stack | Helm chart that installs Prometheus, Alertmanager, Grafana, and related components. |
| Grafana dashboard | Visual page that shows metrics such as rate, errors, latency, CPU, and memory. |
| Alertmanager | Prometheus component that routes firing alerts to receivers such as Mailpit. |
| Mailpit | Local e-mail testing tool used to receive alert e-mail. |
| Runbook | Step-by-step incident guide with symptoms, diagnosis, mitigation, and escalation. |
| Rolling update | Deployment strategy that replaces pods gradually. Prod uses `maxUnavailable: 0`. |
| Rollback | Returning a Helm release to a previous known-good revision. |
| k6 | Load-test tool used to prove latency and zero failed requests. |
| Trivy | Scanner for image vulnerabilities and Kubernetes misconfigurations. |
| Gitleaks | Secret scanner that blocks committed credentials. |
| Dependabot | GitHub tool that opens dependency update pull requests. |
| GHCR | GitHub Container Registry for ShipIt images. |
| Branch protection | GitHub rule that blocks unsafe merges to `main`. |
| Valkey | Permissive Redis-compatible cache option for Should scope. |
| SLO | Service Level Objective, such as 99.5% availability. |
| DORA | Delivery metrics such as deployment frequency and change failure rate. |
| NetworkPolicy | Kubernetes rule that controls pod network traffic. |
| Calico | CNI option that enforces NetworkPolicy in kind. |
| Argo CD | GitOps controller that syncs cluster state from Git. |
| Loki | Grafana log store used in the standard profile. |
| Alloy | Grafana collector for logs and telemetry. It replaces Promtail. |

## Official documentation links

| Tool | Official documentation |
|---|---|
| Python | https://www.python.org/doc/ |
| uv | https://docs.astral.sh/uv/ |
| Ruff | https://docs.astral.sh/ruff/ |
| mypy | https://mypy.readthedocs.io/ |
| pytest | https://docs.pytest.org/ |
| coverage.py | https://coverage.readthedocs.io/ |
| pre-commit | https://pre-commit.com/ |
| pip-audit | https://pypi.org/project/pip-audit/ |
| FastAPI | https://fastapi.tiangolo.com/ |
| Pydantic | https://docs.pydantic.dev/ |
| SQLAlchemy | https://docs.sqlalchemy.org/ |
| Alembic | https://alembic.sqlalchemy.org/ |
| psycopg | https://www.psycopg.org/ |
| Uvicorn | https://uvicorn.dev/ |
| Gunicorn | https://docs.gunicorn.org/ |
| PostgreSQL | https://www.postgresql.org/docs/ |
| Docker Engine and Compose | https://docs.docker.com/ |
| GitHub Actions | https://docs.github.com/actions |
| GHCR | https://docs.github.com/packages/working-with-a-github-packages-registry/working-with-the-container-registry |
| Dependabot | https://docs.github.com/code-security/dependabot |
| kind | https://kind.sigs.k8s.io/ |
| Kubernetes and kubectl | https://kubernetes.io/docs/ |
| Helm | https://helm.sh/docs/ |
| Terraform | https://developer.hashicorp.com/terraform/docs |
| Terraform Helm provider | https://registry.terraform.io/providers/hashicorp/helm/latest/docs |
| Terraform kind provider | https://registry.terraform.io/providers/tehcyx/kind/latest/docs |
| Gateway API | https://gateway-api.sigs.k8s.io/ |
| Envoy Gateway | https://gateway.envoyproxy.io/ |
| Prometheus | https://prometheus.io/docs/ |
| kube-prometheus-stack | https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack |
| Grafana | https://grafana.com/docs/grafana/latest/ |
| Alertmanager | https://prometheus.io/docs/alerting/latest/alertmanager/ |
| Mailpit | https://mailpit.axllent.org/ |
| Trivy | https://trivy.dev/docs/latest/ |
| Gitleaks | https://gitleaks.io/ |
| hadolint | https://github.com/hadolint/hadolint |
| ShellCheck | https://www.shellcheck.net/ |
| kubeconform | https://kubeconform.mandragor.org/ |
| k6 | https://grafana.com/docs/k6/latest/ |
| Valkey | https://valkey.io/docs/ |
| Redis | https://redis.io/docs/latest/ |
| Argo CD | https://argo-cd.readthedocs.io/ |
| Loki | https://grafana.com/docs/loki/latest/ |
| Grafana Alloy | https://grafana.com/docs/alloy/latest/ |
| metrics-server | https://github.com/kubernetes-sigs/metrics-server |
| Calico | https://docs.tigera.io/calico/latest/about/ |
| Sealed Secrets | https://github.com/bitnami-labs/sealed-secrets |
| SOPS | https://getsops.io/docs/ |
| Syft | https://github.com/anchore/syft |
| bats-core | https://bats-core.readthedocs.io/ |
| TFLint | https://github.com/terraform-linters/tflint |
| Jenkins | https://www.jenkins.io/doc/ |

## Free learning resources

| Topic | Resource |
|---|---|
| Linux basics | https://ubuntu.com/tutorials/command-line-for-beginners |
| GitHub skills | https://skills.github.com/ |
| Docker getting started | https://docs.docker.com/get-started/ |
| Kubernetes basics | https://kubernetes.io/docs/tutorials/kubernetes-basics/ |
| Helm getting started | https://helm.sh/docs/intro/quickstart/ |
| Terraform tutorials | https://developer.hashicorp.com/terraform/tutorials |
| Prometheus getting started | https://prometheus.io/docs/prometheus/latest/getting_started/ |
| Grafana fundamentals | https://grafana.com/tutorials/ |
| GitHub Actions tutorial | https://docs.github.com/actions/tutorials |
| AWS fundamentals | https://docs.aws.amazon.com/whitepapers/latest/aws-overview/introduction.html |

## Suggested reading order

1. Read the ShipIt overview, users, FRs, NFRs, and platform specification.
2. Learn Linux commands before Docker debugging.
3. Learn Docker images, containers, volumes, and networks before Kubernetes.
4. Learn Kubernetes Pods, Deployments, Services, StatefulSets, probes, and namespaces.
5. Learn Gateway API and Envoy Gateway routing for `.localhost` hosts.
6. Learn Helm chart, values, release, and rollback concepts.
7. Learn Terraform state, plan, apply, variables, modules, providers, and drift.
8. Learn Prometheus metrics, Grafana dashboards, Alertmanager, RED, USE, SLI, SLO, and SLA.
9. Learn Trivy, Gitleaks, Dependabot, branch protection, and secure image practices.
10. Learn AWS basics after local ShipIt works, then map local components to AWS services.

[Back to README](../README.md)
