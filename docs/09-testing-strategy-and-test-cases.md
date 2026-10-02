# Testing strategy and test cases

Purpose: This document defines the ShipIt test strategy, tools, environments, coverage gates, concrete test catalog, traceability, performance conditions, defect fields, and release criteria.

## Test strategy

ShipIt uses a platform-heavy test pyramid. Fast app and script tests run first. Infrastructure validation and smoke tests protect deployments. k6 and reliability drills prove production behavior.

| Level | Approximate count | Main tools | Scope |
|---|---:|---|---|
| Unit tests | 10 | pytest 9.1.x, pytest-cov 7.1.x, Ruff 0.16.x | URL validation, code generation, config parsing, metrics helpers. |
| Integration tests | 8 | pytest 9.1.x, Testcontainers 4.15.x, PostgreSQL 18.x | API plus PostgreSQL, readiness, redirect states. |
| API and smoke tests | 4 | pytest or smoke script, curl-like checks | Health, create response, optional alias, and optional delete. |
| CLI and script tests | 9 | ShellCheck 0.11.x, help checks, bats-core 1.14.x when implemented | Bootstrap, smoke, backup, restore, and secret scripts. |
| Infrastructure and security validation | 29 | hadolint 2.15.x, Helm 4.x, kubeconform 0.8.x, Terraform 1.16.x, Trivy 0.75.x, Gitleaks 8.30.x | Image, Kubernetes, Helm, Terraform, secrets, PDB, and vulnerabilities. |
| Observability, performance, reliability, and docs | 15 | k6 2.x, Kubernetes 1.37.x, Grafana 13.x, Mailpit 1.31.x | Alerts, p95 latency, rolling update, rollback, pod kill, runbooks, and AWS mapping. |

## Tools and reference versions

| Tool | Version line | Used for |
|---|---|---|
| Python | 3.12.x | App runtime and tests. |
| FastAPI | 0.142.x | API implementation under test. |
| PostgreSQL | 18.x | ShortLink database. |
| Docker Engine | 29.x | Image and local runtime. |
| Docker Compose | v5.x | Local app plus database stack. |
| pytest | 9.1.x | App unit and integration tests. |
| pytest-cov / coverage | 7.1.x / 7.16.x | Coverage gates. |
| Ruff | 0.16.x | Lint and format gate. |
| mypy | 2.4.x | Typed app package gate. |
| ShellCheck | 0.11.x | Script gate. |
| bats-core | 1.14.x | Script behavior tests. |
| hadolint | 2.15.x | Container build file lint. |
| Trivy | 0.75.x | Image and config scan gates. |
| Gitleaks | 8.30.x | Secret scan. |
| kind | 0.33.x | Local Kubernetes cluster. |
| Kubernetes / kubectl | 1.37.x | Runtime and validation target. |
| Helm | 4.x | Chart lint, render, install, test, and rollback. |
| Terraform | 1.16.x | Platform provisioning validation. |
| kubeconform | 0.8.x | Rendered manifest validation. |
| Envoy Gateway | 1.9.x | Gateway API routing tests. |
| kube-prometheus-stack | chart 91.x | Metrics, dashboards, alerts. |
| Mailpit | 1.31.x | Alert e-mail test receiver. |
| k6 | 2.x | Performance and rolling-update drills. |

## Test environments

| Environment | Purpose | Data | Required before use |
|---|---|---|---|
| Local unit | Fast feedback on developer laptop. | In-memory fixtures and fake generated codes. | Python 3.12.x and locked dependencies installed. |
| Compose dev | App and PostgreSQL integration. | Seed links `Ab3kP91`, `old2026`, and `gone999`. | Docker Compose v5.x available. |
| kind dev | Helm dev release and routing. | Dev database only; host `shortlink-dev.localhost`. | Terraform apply or equivalent platform install complete. |
| kind prod | Release, smoke, alerts, and drills. | Prod database only; host `shortlink.localhost`. | 2 app replicas ready and monitoring installed. |
| CI pull request | Required gates. | Synthetic fixtures and rendered manifests. | Branch opened against `main`. |
| Standard performance | k6 and dashboard proof. | At least 1,000 active links and 100 missing-code requests. | 16 GB profile, prod 2 replicas, Prometheus ready. |

## Test data strategy

| Data type | Source | Exact values | Rule |
|---|---|---|---|
| Valid target URL | Fixture | URL value `https://example.in/careers` | Used for successful create and redirect tests. |
| Missing code | Fixture | `missing1` | MUST return 404. |
| Expired code | Seed row | `old2026` expired at `2026-10-01T00:00:00+05:30` | MUST return 410. |
| Deleted code | Seed row | `gone999` with `deleted_at` set | MUST return 410. |
| Invalid scheme URL | Fixture | `ftp://example.in/file` | MUST return 422. |
| Userinfo URL | Fixture | HTTPS URL with user name `demo-user` at `example.in/careers` | Any URL with a user-info part MUST return 422. |
| Long URL | Generated fixture | 2049 characters under `https://example.in/` | MUST return 422. |
| k6 valid redirects | Seed set | Existing codes only | Expected result is 302 with correct `Location`; redirect following is disabled. |
| k6 missing redirects | Seed set | `miss` prefix codes | Expected result is 404 and is counted as an expected response. |
| Backup fixture | Local ignored backup | Backup file `shortlink-prod-20261002-103000.dump` | Restore keeps the backup file. |
| Recorded alert | Prometheus evidence | `ShortLinkTargetDown` in prod | Mailpit receives at least 1 matching e-mail. |
| Latency alert fixture | Test-only delay | 600 ms delay for at least 10 minutes | High-latency alert fires with severity warning. |
| Crash-loop alert fixture | Pod restart evidence | At least 3 restarts within 10 minutes, sustained for 5 minutes | Crash-loop alert fires with severity critical. |

Students MAY record HTTP responses for tests. Recorded responses MUST use fictional domains and MUST NOT contain secrets.

## Coverage thresholds

| Scope | Line coverage | Branch coverage | Measured packages | Exclusions | CI action |
|---|---:|---:|---|---|---|
| ShortLink application | 80% minimum | 70% minimum | App packages under `app/shortlink` | Migrations, generated files, and `__main__` blocks when documented | Fail pull request below either threshold. |
| Script tests | Not measured by Python coverage | Not measured by Python coverage | Required Bash scripts | bats is excluded unless FR-TEST-01 is implemented | Fail on ShellCheck findings or missing help text. bats assertions are required only when FR-TEST-01 is implemented. |
| Infrastructure validation | Not a coverage metric | Not a coverage metric | Rendered Helm, Terraform, image, and manifests | None for Must resources | Fail when validators report errors. |

## Test case catalog

| ID | Title | Type | Priority | Linked requirement IDs | Preconditions | Steps | Test data | Expected result |
|---|---|---|---|---|---|---|---|---|
| TC-UT-001 | Accept valid HTTP target URL | UT | Must | FR-APP-01, BR-02 | URL validator runs in isolation. | Validate the HTTP careers URL. | `http://example.in/jobs` | Validator accepts the value and returns scheme `http` with host `example.in`. |
| TC-UT-002 | Reject FTP target URL | UT | Must | FR-APP-01, BR-02 | URL validator can report a bad scheme. | Validate the FTP file URL. | `ftp://example.in/file` | Validator rejects it and returns `target-url-invalid` for the scheme check. |
| TC-UT-003 | Reject URL with userinfo | UT | Must | FR-APP-01, BR-02 | URL validator can inspect username and password parts. | Validate a HTTPS URL that contains a user name. | HTTPS URL with user name `demo-user` at `example.in/careers` | Validator rejects it for userinfo, and the returned message omits the user-info value. |
| TC-UT-004 | Reject URL longer than 2048 | UT | Must | FR-APP-01, BR-02 | URL validator can count total characters. | Validate a target URL whose total length is 2049. | `https://example.in/` plus enough `a` characters to length 2049 | Validator rejects it and names the 2048-character maximum. |
| TC-UT-005 | Generate seven-character Base62 code | UT | Must | FR-APP-01, BR-01 | Code generator is seeded or controlled for test. | Generate one code. | Random source returns allowed indexes. | Code length is 7 and every character is in `0123456789ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz`. |
| TC-UT-006 | Stop after five code collisions | UT | Must | FR-APP-01, BR-01 | Repository mock reports duplicate code five times. | Call create-code workflow once. | Duplicate code `Ab3kP91` for all attempts. | Workflow raises or returns `code-space-retry-exhausted` after exactly 5 total generation attempts. |
| TC-UT-007 | Build create response fields | UT | Must | FR-APP-01, BR-03 | Link object exists in memory. | Serialize create response. | id `11111111-2222-4333-8444-555555555555`, code `Ab3kP91` | Response has exactly `id`, `code`, `short_url`, `target_url`, and `created_at`. |
| TC-UT-008 | Map missing database config to startup failure | UT | Must | FR-APP-04, BR-09 | Config loader is callable. | Load config without `DATABASE_URL`. | `APP_ENV=prod` only | Loader fails with `config-missing-database-url`. |
| TC-UT-009 | Produce JSON request log fields | UT | Must | FR-APP-04, BR-08 | Logger can receive request context. | Log one successful create request. | request ID `req-demo-001`, status 201 | One JSON object contains `timestamp`, `level`, `app_env`, `request_id`, `method`, `path`, `status_code`, `duration_ms`. |
| TC-UT-010 | Do not log secret values | UT | Must | FR-APP-04, NFR-SEC-03 | Logger receives config error context. | Log startup failure with database URL present in memory. | A local placeholder password value | Captured log does not contain the placeholder value. |
| TC-IT-001 | Create link stores row | IT | Must | FR-APP-01, BR-03 | PostgreSQL 18.x test database is empty. | POST create endpoint once. | JSON body `{"target_url":"https://example.in/careers"}` | API returns 201; database has 1 row with target URL and a 7-character code. |
| TC-IT-002 | Invalid URL creates no row | IT | Must | FR-APP-01, BR-02 | PostgreSQL test database is empty. | POST create endpoint once. | JSON body `{"target_url":"ftp://example.in/file"}` | API returns 422 with `target-url-invalid`; database row count remains 0. |
| TC-IT-003 | Active code redirects | IT | Must | FR-APP-02, BR-05 | Seed row code `Ab3kP91` is active. | GET `/{code}` once. | `Ab3kP91` | API returns 302 with `Location: https://example.in/careers`. |
| TC-IT-004 | Missing code returns 404 | IT | Must | FR-APP-02, BR-04 | Database has no row for requested short code. | Request `GET /missing1`. | `missing1` | Missing lookup returns 404 with JSON error code `short-code-not-found`. |
| TC-IT-005 | Expired code returns 410 | IT | Must | FR-APP-02, BR-04 | Seed row `old2026` expired before the test date. | Request `GET /old2026` on 2026-10-02. | `old2026` | Expired lookup returns 410 with JSON error code `short-code-expired`. |
| TC-IT-006 | Deleted code returns 410 | IT | Must | FR-APP-02, BR-04 | Seed row `gone999` has a non-null deletion timestamp. | Request `GET /gone999`. | `gone999` | Deleted lookup returns 410 with JSON error code `short-code-deleted`. |
| TC-IT-007 | Readiness fails when database is down | IT | Must | FR-APP-03, BR-24 | App runs with PostgreSQL stopped. | GET `/readyz`. | No request body. | API returns 503 with status `not_ready` and error `database-not-ready`. |
| TC-IT-008 | Metrics increment after redirect | IT | Must | FR-APP-03, BR-07 | Seed active code exists and metrics endpoint is enabled. | Scrape metrics, GET active code, scrape metrics again. | `Ab3kP91` | `shortlink_redirects_total` increases by exactly 1. |
| TC-API-001 | Health endpoint reports process health | API | Must | FR-APP-03, BR-11 | App process is running. | GET `/healthz`. | No request body. | API returns 200 with JSON field `status` equal to `ok`. |
| TC-API-002 | Create response uses environment base URL | API | Must | FR-APP-01, BR-03 | App has `BASE_URL=http://shortlink.localhost:8080`. | POST create endpoint. | Target URL `https://example.in/careers` | `short_url` starts with `http://shortlink.localhost:8080/` and ends with the returned code. |
| TC-API-003 | Optional duplicate alias conflict | API | Should | FR-APP-05, BR-39 | Should alias feature is implemented and alias `team-demo` exists. | POST create endpoint with same alias. | alias `team-demo` | API returns 409 and existing row is unchanged. |
| TC-API-004 | Optional delete requires API key | API | Should | FR-APP-05 | Should delete feature is implemented. | Send delete request without `X-API-Key`. | Short code `Ab3kP91` | API returns 401 and the `Ab3kP91` link remains active. |
| TC-CLI-001 | Bootstrap recommends lite on 8 GB | CLI | Must | FR-BASH-01, NFR-HW-02 | Host memory is mocked or actual 8 GB. | Run bootstrap in check-only mode. | Arguments `--profile lite --check-only` | Exit 0 or 1 by prerequisites; output contains `lite profile` and lists missing tools. |
| TC-CLI-002 | Bootstrap rejects unknown profile | CLI | Must | FR-BASH-01 | Script file exists. | Run bootstrap with invalid profile. | `--profile tiny` | Exit code 2 and usage text is printed. |
| TC-CLI-003 | Smoke test passes against prod | CLI | Must | FR-BASH-01 | Prod host is deployed and ready. | Run smoke test with prod base URL. | Base URL `http://127.0.0.1:8080` with host `shortlink.localhost` | Exit code 0; output names health, readiness, create, redirect, and metrics checks as passed. |
| TC-CLI-004 | Smoke test fails on wrong host | CLI | Must | FR-BASH-01 | No service listens on wrong host. | Run smoke test with wrong base URL. | `http://wrong.localhost` | Exit code 1 and output names the first failed endpoint. |
| TC-CLI-005 | Backup creates timestamped file | CLI | Must | FR-BASH-01, BR-14 | Prod database is reachable and backup directory exists. | Run backup for prod. | Arguments `--env prod --output-dir backups --retention-days 7` | Exit code 0; file name matches `shortlink-prod-YYYYMMDD-HHMMSS.dump`. |
| TC-CLI-006 | Backup rotates old file | CLI | Must | FR-BASH-01, BR-14 | Backup directory contains one prod file older than 7 days. | Run backup for prod. | old file `shortlink-prod-20260920-010000.dump` | Exit code 0; old file is removed and new file exists. |
| TC-CLI-007 | Restore requires file argument | CLI | Must | FR-BASH-01 | Restore script exists. | Run restore without file. | `--env prod` | Exit code 2; usage text is printed; database is not changed. |
| TC-CLI-008 | Restore keeps backup file | CLI | Must | NFR-REL-03, FR-BASH-01 | Backup file exists with 3 link rows. | Run restore with confirmation. | Backup file `shortlink-prod-20261002-103000.dump` | Exit code 0; target database has 3 link rows and backup file still exists. |
| TC-INFRA-001 | Image stays below size limit | INFRA | Must | FR-CON-01, NFR-PERF-03 | App image has been built in CI. | Inspect final image size. | Image tagged with the short commit SHA | Size is less than 200 MB. |
| TC-INFRA-002 | Image runs as non-root | INFRA | Must | FR-CON-01, NFR-SEC-01 | App image has been built. | Inspect runtime user metadata. | Image built from the pull request commit | Runtime user is not root and not UID 0. |
| TC-INFRA-003 | Image contains no local env secret | INFRA | Must | FR-CON-01, NFR-SEC-03 | Local env contains fake secret before build. | Build image and inspect history or filesystem. | Fake variable `FAKE_SECRET=shipit-demo-secret` | Secret text is not found. |
| TC-INFRA-004 | Compose starts app and PostgreSQL healthy | INFRA | Must | FR-CON-02, BR-12 | Docker Engine 29.x is running. | Start Compose stack and wait for health. | Services `app` and `postgres` | Both services report healthy within the documented timeout. |
| TC-INFRA-005 | Compose fails clearly without database config | INFRA | Must | FR-CON-02, BR-09 | Compose env file lacks database URL or required database parts. | Start Compose stack. | Missing database config | App is unhealthy and logs `config-missing-database-url`. |
| TC-INFRA-006 | Required checks list all mandatory gates | INFRA | Must | FR-CI-01, BR-15 | Pull request or push to `main` runs CI. | Read CI check names. | PR and `main` commit with app change | Required checks include Ruff, mypy, hadolint, ShellCheck, pytest coverage, image build, Trivy, Gitleaks, Helm lint, kubeconform, Terraform fmt, Terraform validate. |
| TC-INFRA-007 | Coverage gate blocks 79 percent line coverage | INFRA | Must | FR-CI-01, NFR-MAINT-01 | Test branch reports 79% line coverage. | Run PR workflow. | Coverage report line 79%, branch 75% | Workflow fails because line coverage is below 80%. |
| TC-INFRA-008 | GHCR publish uses immutable version tags | INFRA | Must | FR-CI-02, BR-16 | Main workflow runs for app version `v1.2.0` and commit `abc1234`. | Inspect published image tags and retry the SemVer tag with a different digest. | SemVer `v1.2.0`, SHA `abc1234` | GHCR contains both tags for one digest; reusing `v1.2.0` for another digest is rejected. |
| TC-INFRA-009 | Branch protection blocks failing secret scan | INFRA | Must | FR-CI-03, BR-17 | Branch protection is enabled. | Attempt to merge PR with failing Gitleaks check. | PR status contains failed Gitleaks | GitHub blocks merge. |
| TC-INFRA-010 | Kubernetes namespaces and prod replicas exist | INFRA | Must | FR-K8S-01, BR-18 | Platform apply completed. | List namespaces and prod app pods. | No request body. | Namespaces `shortlink-dev` and `shortlink-prod` exist; prod has exactly 2 ready app pods. |
| TC-INFRA-011 | Gateway routes prod host | INFRA | Must | FR-K8S-02, BR-21 | Envoy Gateway is installed and prod app is ready. | Request prod health through `shortlink.localhost`. | Prod host header | Response is 200 from prod and not from dev. |
| TC-INFRA-012 | Gateway routes dev host separately | INFRA | Must | FR-K8S-02, BR-21 | Dev app is ready and prod metrics are visible. | Request dev health through `shortlink-dev.localhost`. | Dev host header | Response is 200 from dev and prod request count does not increase. |
| TC-INFRA-013 | Missing secret source fails secret creation | INFRA | Must | FR-K8S-03, BR-22 | Secret source file is absent. | Run secret creation procedure. | Missing local secret file | Procedure exits non-zero with message `secret-file-missing`. |
| TC-INFRA-014 | Pod security settings are rendered | INFRA | Must | FR-K8S-04, BR-23 | Helm chart is rendered for prod. | Check rendered app container settings. | Prod values | Container has non-root, read-only root filesystem, drops ALL capabilities, and has requests and limits. |
| TC-INFRA-015 | Readiness removes pod from service | INFRA | Must | FR-K8S-04, BR-24 | One prod pod can be made not ready. | Force readiness endpoint to return 503 and list Service endpoints. | Affected pod name | Affected pod is absent from Service endpoints. |
| TC-INFRA-016 | PostgreSQL resources are separate | INFRA | Must | FR-K8S-05, BR-19 | Dev and prod are installed. | List StatefulSets, PVCs, and Services in both namespaces. | No request body. | Each namespace has one PostgreSQL StatefulSet, one PVC, and one Service; names are environment-specific. |
| TC-INFRA-017 | Helm chart passes lint and test hook | INFRA | Must | FR-HELM-01, BR-27 | Chart exists and release is installed. | Run Helm lint and Helm test. | Chart `shortlink` | Lint exits 0; Helm test verifies `/healthz` and `/readyz`. |
| TC-INFRA-018 | Dev and prod values differ correctly | INFRA | Must | FR-HELM-02, BR-18 | Chart can render both values files. | Render dev values and prod values. | dev and prod values | Dev host is `shortlink-dev.localhost` with 1 replica; prod host is `shortlink.localhost` with 2 replicas. |
| TC-INFRA-019 | Terraform plan rejects unknown profile | INFRA | Must | FR-IAC-02, BR-29 | Terraform root is initialized. | Run Terraform plan with invalid profile. | `profile=unknown` | Plan fails before apply and names valid values `lite` and `standard`. |
| TC-INFRA-020 | Terraform outputs platform endpoints | INFRA | Must | FR-IAC-01, FR-IAC-02 | Terraform apply completed. | Read outputs. | No request body. | Outputs include cluster name, dev host, prod host, Grafana URL, Prometheus URL, Alertmanager URL, and Mailpit URL. |
| TC-INFRA-021 | kubeconform validates rendered manifests | INFRA | Must | FR-CI-01, FR-K8S-01 | Helm renders dev and prod resources. | Run kubeconform over rendered output. | Rendered manifests | 100% of rendered manifests are valid and no resource is skipped silently. |
| TC-INFRA-022 | kubeconform uses CRD schemas | INFRA | Must | FR-CI-01 | Rendered manifests include Gateway API and Prometheus Operator resources. | Run kubeconform with schema locations for official or maintained CRD schemas. | Gateway, HTTPRoute, ServiceMonitor, PrometheusRule | Check fails if a custom kind has no matching schema or is skipped. |
| TC-INFRA-023 | Prod PDB protects voluntary disruption | INFRA | Should | FR-K8S-07, BR-56 | PDB Should item is implemented. | Drain a 1-node cluster, then repeat on a multi-node cluster under k6. | `minAvailable: 1` | 1-node drain is blocked as expected; multi-node drain completes with 0 failed requests. |
| TC-SEC-001 | Trivy blocks critical image finding | SEC | Must | FR-SEC-01, NFR-SEC-02 | Test image report contains one CRITICAL finding. | Run the image vulnerability gate. | CRITICAL OpenSSL finding | CI fails and no GHCR image is pushed. |
| TC-SEC-002 | Trivy blocks HIGH with fix | SEC | Must | FR-SEC-01, BR-36 | Test image report contains HIGH finding with fixed version. | Run the fix-available vulnerability gate. | HIGH package with fix available | CI fails and report names package and fixed version. |
| TC-SEC-003 | Gitleaks blocks committed secret pattern | SEC | Must | Secret hygiene NFR-SEC-03; scan FR-SEC-01; CI FR-CI-01 | Throw-away branch includes a synthetic, never-issued value in the GitHub personal-access-token format. | Run secret scan gate and delete the branch after evidence is saved. | Described fixture only; no real-looking token is written in this specification. | CI fails with a Gitleaks finding for the GitHub PAT rule and the branch is never merged. |
| TC-SEC-004 | Dependabot covers three ecosystems | SEC | Must | FR-SEC-02, BR-37 | Dependency automation is configured. | Review Dependabot coverage. | Ecosystems GitHub Actions, pip, Docker | All three ecosystems are present. |
| TC-OBS-001 | Prometheus target is up | E2E | Must | FR-OBS-01, NFR-OBS-01 | ShortLink prod is deployed with ServiceMonitor. | Open Prometheus targets. | Target label for prod app | ShortLink target is `up` in prod. |
| TC-OBS-002 | Grafana RED panels show traffic | E2E | Must | FR-OBS-02, BR-31 | Generate traffic for 5 minutes. | Open dashboard and filter prod. | Mixed create and redirect traffic | Panels show request rate, 5xx rate, p50, p95, p99, CPU, memory, and pod restarts. |
| TC-OBS-003 | High error rate alert fires | E2E | Must | FR-OBS-03, BR-32 | Prometheus has loaded the error-rate rule. | Generate 5xx rate above 5% for 5 minutes. | 6 failed requests per 100 total | `ShortLinkHighErrorRate` fires with severity critical. |
| TC-OBS-004 | 450 ms p95 does not fire latency alert | E2E | Must | FR-OBS-03, BR-32 | Prometheus has loaded the latency rule. | Generate p95 latency near 450 ms for 10 minutes. | p95 450 ms | `ShortLinkHighLatencyP95` does not fire. |
| TC-OBS-005 | Mailpit receives target down alert | E2E | Must | FR-OBS-04, BR-33 | Alertmanager routes to Mailpit. | Make ShortLink target down for 5 minutes. | Prod target unavailable | Mailpit receives at least 1 matching e-mail with environment `prod`, severity `critical`, and a runbook link; duplicates are allowed. |
| TC-OBS-006 | High latency alert fires | E2E | Must | Alert rule FR-OBS-03; coverage NFR-OBS-02; threshold BR-32 | Prometheus has loaded the latency rule and test delay is enabled. | Inject 600 ms delay for at least 10 minutes. | Prod redirect route with delay | `ShortLinkHighLatencyP95` fires with severity warning. |
| TC-OBS-007 | Crash-loop alert fires | E2E | Must | Restart alert FR-OBS-03; runbook NFR-OBS-02; threshold BR-32 | Prometheus has loaded the crash-loop rule. | Create at least 3 container restarts within 10 minutes and keep the condition for 5 minutes. | Prod app pod restart evidence | `ShortLinkPodCrashLooping` fires with severity critical. |
| TC-REL-001 | Rolling update has zero failed requests | PERF | Must | FR-K8S-06, FR-REL-02, NFR-REL-01 | Prod has 2 replicas and k6 uses existing seeded codes only. | Run k6 with redirect following disabled, deploy new SHA image tag, wait for rollout, stop k6. | 50 VUs on 16 GB standard profile; 20 VUs on 8 GB lite evidence | Existing codes return expected 302 responses, no unexpected status or transport error occurs, and rollout completes. |
| TC-REL-002 | Pod kill recovers within 120 seconds | E2E | Must | FR-REL-02, NFR-REL-02 | Prod has 2 ready app pods. | Delete one app pod and watch replacement. | One selected prod pod | Replacement pod is Ready within 120 seconds. |
| TC-REL-003 | Helm rollback restores previous release | E2E | Must | FR-REL-02, BR-35 | A bad prod release is active and previous revision is healthy. | Roll back to previous revision and run smoke test. | Previous revision number | Smoke test exits 0 and release history shows rolled-back revision. |
| TC-PERF-001 | Redirect p95 stays below 300 ms | PERF | Must | NFR-PERF-01, FR-APP-02 | Prod has 2 ready replicas. | Run redirect k6 scenario with redirect following disabled. | 50 VUs on 16 GB standard profile; 20 VUs on 8 GB lite evidence; expected 302 redirects and expected 404 misses | Standard p95 for valid redirects is below 300 ms; lite results are recorded; unexpected status codes and transport errors are below 1%. |
| TC-PERF-002 | Create p95 stays below 500 ms | PERF | Must | NFR-PERF-02, FR-APP-01 | Standard profile is ready. | Run create-link k6 scenario. | 20 VUs, unique `https://example.in/perf/{n}` URLs | p95 create latency is below 500 ms for 5 minutes. |
| TC-PERF-003 | HPA scales above two replicas | PERF | Should | Scaling behavior FR-SCALE-01 and NFR-SCALE-01 | HPA Should item is implemented. | Run CPU load and watch HPA. | CPU above 70% target | Replica count increases above 2 and later returns to 2. |
| TC-SEC-005 | NetworkPolicy blocks debug pod | SEC | Should | FR-NET-01, NFR-NET-01 | Calico or Cilium and NetworkPolicy are enabled. | Connect to PostgreSQL from app and from unlabeled debug pod. | PostgreSQL service address | App connection succeeds; debug pod connection is denied. |
| TC-E2E-001 | Loki finds request ID | E2E | Should | FR-LOG-01, NFR-LOG-01 | Loki and Alloy are enabled in standard profile. | Send request with ID and search Grafana Explore. | request ID `req-loki-001` | Log line is searchable within 5 minutes. |
| TC-INFRA-024 | Jenkins stages mirror GitHub Actions | INFRA | Should | FR-JENKINS-01, BR-49 | Jenkins explanation exists. | Compare Jenkins stage list with CI gate list. | Gate list from BR-15 | Each GitHub Actions gate maps to one Jenkins stage. |
| TC-DOC-001 | Alert runbooks contain required sections | DOC | Must | FR-REL-01, NFR-OBS-02 | Four runbook files exist. | Review each runbook heading list. | Required sections from document 08 | All four runbooks contain symptoms, impact, diagnosis, mitigation, escalation, and evidence sections. |
| TC-DOC-002 | AWS mapping table covers required rows | DOC | Must | FR-DOC-01, BR-38 | Local-to-AWS table exists. | Check each required local platform item. | kind, GHCR, PVC, Gateway, Alertmanager e-mail, local Terraform state | Every required row has one AWS equivalent and one interview point. |
| TC-CLI-009 | Secret creation reports missing file | CLI | Must | FR-K8S-03, BR-22 | `scripts/create-secrets.sh` exists. | Run create-secrets with a missing file. | `--env prod --file missing.env` | Exit code is 2 and output includes `secret-file-missing`. |

## Functional requirement traceability

| Requirement IDs | Evidence |
|---|---|
| FR-APP-01 | TC-UT-001, TC-UT-002, TC-UT-003, TC-UT-004, TC-UT-005, TC-UT-006, TC-UT-007, TC-IT-001, TC-IT-002, TC-API-002, TC-PERF-002 |
| FR-APP-02 | TC-IT-003, TC-IT-004, TC-IT-005, TC-IT-006, TC-PERF-001 |
| FR-APP-03 | Health and metrics evidence: TC-IT-007, TC-IT-008, TC-API-001 |
| FR-APP-04 | Config and logging evidence: TC-UT-008, TC-UT-009, TC-UT-010 |
| FR-CON-01 | Image safety evidence: TC-INFRA-001, TC-INFRA-002, TC-INFRA-003 |
| FR-CON-02 | Compose evidence: TC-INFRA-004, TC-INFRA-005 |
| FR-BASH-01 | TC-CLI-001, TC-CLI-002, TC-CLI-003, TC-CLI-004, TC-CLI-005, TC-CLI-006, TC-CLI-007, TC-CLI-008 |
| FR-CI-01 | CI validation evidence: TC-INFRA-006, TC-INFRA-007, TC-INFRA-021, TC-INFRA-022, TC-SEC-003 |
| FR-CI-02 | TC-INFRA-008 |
| FR-CI-03 | TC-INFRA-009 |
| FR-K8S-01 | Namespace evidence: TC-INFRA-010, TC-INFRA-021 |
| FR-K8S-02 | Gateway evidence: TC-INFRA-011, TC-INFRA-012 |
| FR-K8S-03 | TC-INFRA-013, TC-CLI-009 |
| FR-K8S-04 | Pod safety evidence: TC-INFRA-014, TC-INFRA-015 |
| FR-K8S-05 | TC-INFRA-016 |
| FR-K8S-06 | TC-REL-001 |
| FR-HELM-01 | TC-INFRA-017 |
| FR-HELM-02 | TC-INFRA-018 |
| FR-IAC-01 | TC-INFRA-020 |
| FR-IAC-02 | Terraform contract evidence: TC-INFRA-019, TC-INFRA-020 |
| FR-OBS-01 | TC-OBS-001 |
| FR-OBS-02 | TC-OBS-002 |
| FR-OBS-03 | Alert-rule evidence: TC-OBS-003, TC-OBS-004, TC-OBS-006, TC-OBS-007 |
| FR-OBS-04 | TC-OBS-005 |
| FR-REL-01 | TC-DOC-001 |
| FR-REL-02 | Drill evidence: TC-REL-001, TC-REL-002, TC-REL-003 |
| FR-SEC-01 | Scan evidence: TC-SEC-001, TC-SEC-002, TC-SEC-003 |
| FR-SEC-02 | TC-SEC-004 |
| FR-DOC-01 | TC-DOC-002 |
| FR-APP-05 | TC-API-003, TC-API-004 |
| FR-GITOPS-01 | Demo step: show Argo CD dev auto-sync and prod manual sync when implemented. |
| FR-LOG-01 | TC-E2E-001 |
| FR-SCALE-01 | TC-PERF-003 |
| FR-NET-01 | TC-SEC-005 |
| FR-KIND-01 | Demo step: list 3 Ready nodes in standard multi-node profile. |
| FR-K8S-07 | TC-INFRA-023 |
| FR-TEST-01 | Demo step: show bats, SBOM, ephemeral kind deploy test, release workflow, yamllint, and TFLint evidence. |
| FR-SECRETS-01 | Demo step: show SOPS or Sealed Secrets encrypted secret evidence. |
| FR-SLO-01 | Demo step: open SLO dashboard with 99.5% availability and 300 ms p95 targets. |
| FR-POST-01 | Demo step: review bad-release postmortem and DORA notes. |
| FR-HELM-03 | Demo step: run helm-unittest for dev host, prod replicas, and security context values. |
| FR-IAC-03 | Demo step: run Terraform test and TFLint evidence. |
| FR-JENKINS-01 | TC-INFRA-024 |
| FR-ANSIBLE-01 | Demo step: Multipass Ubuntu VM reaches kind version. |
| FR-ROLLOUT-01 | Demo step: canary receives 10% traffic before promotion. |
| FR-SIGN-01 | Demo step: unsigned image is rejected in a test namespace. |
| FR-OTEL-01 | Demo step: show one trace from API handler to database call. |
| FR-AWS-01 | Demo step: show cost warning and destroy evidence. |
| FR-DBOP-01 | Demo step: compare StatefulSet design with CloudNativePG in ADR. |

## NFR traceability

| NFR ID | Evidence |
|---|---|
| NFR-PERF-01 | TC-PERF-001 |
| NFR-PERF-02 | TC-PERF-002 |
| NFR-PERF-03 | TC-INFRA-001 |
| NFR-REL-01 | TC-REL-001 |
| NFR-REL-02 | TC-REL-002 |
| NFR-REL-03 | TC-CLI-008 |
| NFR-SEC-01 | Runtime security evidence: TC-INFRA-002, TC-INFRA-014 |
| NFR-SEC-02 | TC-SEC-001, TC-SEC-002 |
| NFR-SEC-03 | Secret-hygiene evidence: TC-UT-010, TC-INFRA-003, TC-SEC-003 |
| NFR-SEC-04 | TC-SEC-004 |
| NFR-OBS-01 | TC-OBS-001, TC-OBS-002 |
| NFR-OBS-02 | Alert coverage evidence: TC-OBS-003, TC-OBS-004, TC-OBS-005, TC-OBS-006, TC-OBS-007, TC-DOC-001 |
| NFR-MAINT-01 | TC-INFRA-007 |
| NFR-MAINT-02 | TC-CLI-001, TC-CLI-002, TC-CLI-003, TC-CLI-004, TC-CLI-005, TC-CLI-006, TC-CLI-007, TC-CLI-008 |
| NFR-PORT-01 | Demo step: clean-clone setup on Windows WSL2 Ubuntu, macOS, or Linux. |
| NFR-HW-01 | Demo step: trainer pre-check on 8 GB lite profile. |
| NFR-HW-02 | TC-CLI-001 |
| NFR-CI-01 | CI-enforcement evidence: TC-INFRA-006, TC-INFRA-007, TC-INFRA-009, TC-INFRA-022 |
| NFR-DOC-01 | Demo step: start Must system from README in 10 steps or fewer. |
| NFR-LIC-01 | Demo step: ADR and licence notes for Terraform, OpenTofu, Valkey, Bitnami, and Mailpit choices. |
| NFR-PRIV-01 | TC-UT-003 and data review using only fictional domains. |
| NFR-SCALE-01 | TC-PERF-003 |
| NFR-NET-01 | TC-SEC-005 |
| NFR-LOG-01 | TC-E2E-001 |
| NFR-SLO-01 | Demo step: SLO dashboard and burn-rate alert evidence. |

## Business rule evidence matrix

| Business rule IDs | Evidence |
|---|---|
| BR-01 | Code-generation evidence: TC-UT-005, TC-UT-006, TC-IT-001 |
| BR-02 | TC-UT-001, TC-UT-002, TC-UT-003, TC-UT-004, TC-IT-002 |
| BR-03 | TC-UT-007, TC-API-002 |
| BR-04 | Lookup-state evidence: TC-IT-004, TC-IT-005, TC-IT-006 |
| BR-05 | TC-IT-003 |
| BR-06 | TC-IT-008 and redirect dashboard review. |
| BR-07 | TC-IT-008, TC-OBS-002 and label review. |
| BR-08 | TC-UT-009 |
| BR-09 | TC-UT-008, TC-INFRA-005 |
| BR-10 | Container-rule evidence: TC-INFRA-001, TC-INFRA-002, TC-INFRA-003 |
| BR-11 | TC-API-001 |
| BR-12 | Compose health evidence: TC-INFRA-004, TC-INFRA-005 |
| BR-13 | TC-CLI-001 through TC-CLI-008 plus ShellCheck gate. |
| BR-14 | TC-CLI-005, TC-CLI-006 |
| BR-15 | Gate-list evidence: TC-INFRA-006, TC-INFRA-007 |
| BR-16 | TC-INFRA-008 |
| BR-17 | TC-INFRA-009 |
| BR-18 | Namespace-values evidence: TC-INFRA-010, TC-INFRA-018 |
| BR-19 | TC-INFRA-016 |
| BR-20 | Gateway-schema evidence: TC-INFRA-011, TC-INFRA-012, TC-INFRA-022 |
| BR-21 | Host-routing evidence: TC-INFRA-011, TC-INFRA-012, TC-INFRA-018 |
| BR-22 | TC-INFRA-013, TC-CLI-009 |
| BR-23 | TC-INFRA-014 |
| BR-24 | TC-IT-007, TC-INFRA-015 |
| BR-25 | TC-INFRA-016 and image-source review. |
| BR-26 | TC-REL-001 |
| BR-27 | TC-INFRA-017 |
| BR-28 | ADR-001 review and Terraform validation. |
| BR-29 | Terraform-scope evidence: TC-INFRA-019, TC-INFRA-020 |
| BR-30 | TC-OBS-001 and retention review. |
| BR-31 | TC-OBS-002 |
| BR-32 | Alert-threshold evidence: TC-OBS-003, TC-OBS-004, TC-OBS-006, TC-OBS-007 |
| BR-33 | TC-OBS-005 |
| BR-34 | TC-DOC-001. |
| BR-35 | Reliability-drill evidence: TC-REL-001, TC-REL-002, TC-REL-003 |
| BR-36 | TC-SEC-001, TC-SEC-002 |
| BR-37 | TC-SEC-004 |
| BR-38 | FR-DOC-01 viva demo step. |
| BR-39 | TC-API-003, TC-API-004 |
| BR-40 | FR-GITOPS-01 demo step. |
| BR-41 | TC-E2E-001 |
| BR-42 | TC-PERF-003 |
| BR-43 | TC-SEC-005 |
| BR-44 | FR-KIND-01 demo step. |
| BR-45 | FR-TEST-01, FR-HELM-03, and FR-IAC-03 demo steps. |
| BR-46 | FR-SECRETS-01 demo step. |
| BR-47 | FR-SLO-01 demo step. |
| BR-48 | FR-POST-01 demo step. |
| BR-49 | TC-INFRA-024 |
| BR-50 | FR-ANSIBLE-01 demo step. |
| BR-51 | FR-ROLLOUT-01 demo step. |
| BR-52 | FR-SIGN-01 demo step. |
| BR-53 | FR-OTEL-01 demo step. |
| BR-54 | FR-AWS-01 demo step. |
| BR-55 | FR-DBOP-01 demo step. |
| BR-56 | TC-INFRA-023 |

## Performance-test conditions

| Item | Required condition |
|---|---|
| Hardware | Standard 16 GB profile for the 50-VU acceptance run; 8 GB lite profile records the same script at 20 VUs. |
| Cluster | kind 0.33.x, Kubernetes 1.37.x, prod namespace has 2 ready app replicas. |
| Data volume | At least 1,000 active links, 1 expired link, 1 deleted link, and 100 missing-code samples. |
| Warm-up | 2 minutes of low-rate traffic before measured run. |
| Redirect run | 50 virtual users for 5 minutes on standard, or 20 virtual users on lite evidence; redirect following disabled; valid redirects and expected misses are measured separately. |
| Create run | 20 virtual users for 5 minutes; each request uses a unique `https://example.in/perf/{n}` URL. |
| Thresholds | Valid redirect p95 below 300 ms; create p95 below 500 ms; unexpected status or transport-error rate below 1%; rolling update 0 failed requests. |
| Evidence | Save k6 summary, dashboard timestamp, Git SHA, image tag, profile, and cluster node count. |

## Defect report fields

| Field | Required content |
|---|---|
| Title | Short failure summary with area, such as `prod smoke redirect returns 404`. |
| Requirement IDs | FR, BR, or NFR IDs affected. |
| Environment | Local unit, Compose dev, kind dev, kind prod, CI, or standard performance. |
| Build reference | Git SHA, image tag, and Helm release revision when applicable. |
| Steps to reproduce | Numbered steps with exact command or HTTP operation names. |
| Expected result | Exact status, output, metric, or resource state. |
| Actual result | Exact observed status, output, metric, or resource state. |
| Evidence | Log excerpt, screenshot description, k6 summary, CI URL, or dashboard timestamp. |
| Severity | Critical, high, medium, or low. |
| Owner | Student role responsible for fixing it. |
| Fix notes | Summary of code, config, or documentation change. |
| Regression test | Test case ID added or updated. |

## Entry criteria

- The student repository exists and has the expected folders.
- Python dependencies install from the lockfile.
- Docker Engine and Docker Compose run locally.
- The app can start in local unit or Compose mode.
- The required script files exist and show help text.
- The Helm chart, Terraform root, and CI workflow design are present before infrastructure tests run.
- Test data uses only fictional domains and no real personal data.

## Exit criteria

- All Must test cases pass or have trainer-approved evidence.
- Pull request CI is green and branch protection blocks unsafe merges.
- App coverage is at least 80% line and 70% branch.
- kubeconform validates 100% of rendered Must manifests.
- Helm lint, Terraform validate, ShellCheck, Ruff, Trivy, and Gitleaks pass.
- Smoke tests pass after each dev and prod deployment.
- k6 proves redirect p95 below 300 ms and rolling update 0 failed requests.
- Reliability evidence includes rolling update, rollback, pod kill, and backup restore.
- Every Must FR, BR, and NFR has a linked test case or named demo step.

[Back to README](../README.md)
