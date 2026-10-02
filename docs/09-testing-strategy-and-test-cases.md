# Testing strategy and test cases

Purpose: This document defines the ShipIt test strategy, tools, environments, coverage gates, concrete test catalog, traceability, performance conditions, defect fields, and release criteria.

## Test strategy

ShipIt uses a platform-heavy test pyramid. Fast app and script tests run first. Infrastructure validation and smoke tests protect deployments. k6 and reliability drills prove production behavior.

Keep the original minimum of **45 meaningful cases** (this catalog remains above it). Version 1.1 replaces visualization/cloud evidence with backend/local DevOps checks. TC-LOCAL-001 through TC-LOCAL-004 are mandatory for both stages.

| Level | Approximate count | Main tools | Scope |
|---|---:|---|---|
| Unit tests | 10 | pytest 9.1.x, pytest-cov 7.1.x, Ruff 0.16.x | URL validation, code generation, config parsing, metrics helpers. |
| Integration tests | 8 | pytest 9.1.x, Testcontainers 4.15.x, PostgreSQL 18.x | API plus PostgreSQL, readiness, redirect states. |
| API and smoke tests | 4 | pytest or smoke script, curl-like checks | Health, create response, optional alias, and optional delete. |
| CLI and script tests | 9 | ShellCheck 0.11.x, help checks, bats-core 1.14.x when implemented | Bootstrap, smoke, backup, restore, and secret scripts. |
| Infrastructure and security validation | 29 | hadolint 2.15.x, Helm 4.x, kubeconform 0.8.x, Terraform 1.16.x, Trivy 0.75.x, Gitleaks 8.30.x | Image, Kubernetes, Helm, Terraform, secrets, PDB, and vulnerabilities. |
| Observability, performance, reliability, and docs | 15 | k6 2.x, Kubernetes 1.37.x, Prometheus API/CLI, Mailpit 1.31.x | Metric/alert evidence, latency, rollout/rollback/pod kill, runbooks, and interview theory. |
| Local acceptance | 4 (each stage) | Make entrypoints, pytest/CLI/API checks | Compose first, one-node kind next, offline demo, persistence, and clear failures. |

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
| kube-prometheus-stack | chart 91.x | Prometheus metrics/query/alert evidence; no dashboard authoring. |
| Mailpit | 1.31.x | Alert e-mail test receiver. |
| k6 | 2.x | Performance and rolling-update drills. |

## Test environments

| Environment | Purpose | Data | Required before use |
|---|---|---|---|
| Local unit | Fast feedback on developer laptop. | In-memory fixtures and fake generated codes. | Python 3.12.x and locked dependencies installed. |
| Compose dev | First runnable stage and app/PostgreSQL integration. | Local acceptance seed `Ab3kP91`; isolated test fixtures add expired/deleted rows. | Cached local app/PostgreSQL images, ignored credentials, Compose v5.x. |
| kind dev | Helm dev release and routing. | Dev database only; host `shortlink-dev.localhost`. | Terraform apply or equivalent platform install complete. |
| kind prod | Second-stage release, smoke, alerts, drills. | Prod only active; host `shortlink.localhost` at loopback 8081. | Local images loaded; lite 1/standard 2 app replicas; monitoring ready; inactive dev 0. |
| CI pull request | Required gates. | Synthetic fixtures and rendered manifests. | Branch opened against `main`. |
| Standard performance | k6 and saved metric/API proof. | At least 1,000 active links and 100 missing-code requests. | 16 GB, prod 2 replicas; lite records same script at 20 VUs/1 replica. |

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

Store recorded API/Prometheus/alert source fixtures and rule-series fixtures in the student tests, without secrets. Local acceptance uses document 06's one-row seed; other catalogs use isolated test databases and may add extra rows. No runtime test may contact fictional target URLs or external DNS/registry services. Disable redirect following. Prepare dependencies, provider/chart/schema caches and images first, then deny external egress while allowing loopback/container/cluster networking. GitHub/CI/GHCR tests are separate connected exercises.

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
| TC-IT-008 | Metrics increment after redirect | IT | Must | FR-APP-03, BR-07 | Seed code exists and metrics endpoint is enabled. | Scrape, GET seed without following redirect, scrape again. | `Ab3kP91` | Successful `shortlink_redirects_total{status="redirected"}` series increases exactly 1; no external request occurs. |
| TC-API-001 | Health endpoint reports process health | API | Must | FR-APP-03, BR-11 | App process is running. | GET `/healthz`. | No request body. | API returns 200 with JSON field `status` equal to `ok`. |
| TC-API-002 | Create response uses environment base URL | API | Must | FR-APP-01, BR-03 | kind prod has `BASE_URL=http://shortlink.localhost:8081`; separate Compose fixture uses `http://127.0.0.1:8080`. | POST in each isolated stage. | Fictional `https://example.in/careers` | Each `short_url` uses the selected base URL and returned code; no target fetch. |
| TC-API-003 | Optional duplicate alias conflict | API | Should | FR-APP-05, BR-39 | Should alias feature is implemented and alias `team-demo` exists. | POST create endpoint with same alias. | alias `team-demo` | API returns 409 and existing row is unchanged. |
| TC-API-004 | Optional delete requires API key | API | Should | FR-APP-05 | Should delete feature is implemented. | Send delete request without `X-API-Key`. | Short code `Ab3kP91` | API returns 401 and the `Ab3kP91` link remains active. |
| TC-CLI-001 | Bootstrap recommends lite on 8 GB | CLI | Must | FR-BASH-01, NFR-HW-02 | Host memory is mocked or actual 8 GB. | Run bootstrap in check-only mode. | Arguments `--profile lite --check-only` | Exit 0 or 1 by prerequisites; output contains `lite profile` and lists missing tools. |
| TC-CLI-002 | Bootstrap rejects unknown profile | CLI | Must | FR-BASH-01 | Script file exists. | Run bootstrap with invalid profile. | `--profile tiny` | Exit code 2 and usage text is printed. |
| TC-CLI-003 | Smoke test passes against prod | CLI | Must | FR-BASH-01 | Only prod is active and ready. | Run smoke through kind Gateway without following redirects. | `http://127.0.0.1:8081`, Host `shortlink.localhost` | Exit 0; health/readiness/create/302 Location/metrics pass without external egress. |
| TC-CLI-004 | Smoke test fails on wrong host | CLI | Must | FR-BASH-01 | Envoy runs but no HTTPRoute matches wrong Host. | Run smoke with loopback base and wrong Host. | `http://127.0.0.1:8081`, Host `wrong.localhost` | Health returns 404; script exits 1 and names the first failed endpoint without DNS lookup. |
| TC-CLI-005 | Backup creates timestamped file | CLI | Must | FR-BASH-01, BR-14 | Selected active database and ignored backup directory exist. | Back up kind prod; repeat for Compose dev. | `--stage kind --env prod --output-dir .local/backups --retention-days 7` | Exit 0; timestamped `.dump` is created for correct stage/env; no host PostgreSQL port or external service needed. |
| TC-CLI-006 | Backup rotates old file | CLI | Must | FR-BASH-01, BR-14 | Backup directory contains one prod file older than 7 days. | Run backup for prod. | old file `shortlink-prod-20260920-010000.dump` | Exit code 0; old file is removed and new file exists. |
| TC-CLI-007 | Restore requires file argument | CLI | Must | FR-BASH-01 | Restore script exists. | Run restore without file. | `--env prod` | Exit code 2; usage text is printed; database is not changed. |
| TC-CLI-008 | Restore keeps backup file | CLI | Must | NFR-REL-03, FR-BASH-01 | Backup file exists with 3 link rows. | Run restore with confirmation. | Backup file `shortlink-prod-20261002-103000.dump` | Exit code 0; target database has 3 link rows and backup file still exists. |
| TC-INFRA-001 | Image stays below size limit | INFRA | Must | FR-CON-01, NFR-PERF-03 | App image has been built in CI. | Inspect final image size. | Image tagged with the short commit SHA | Size is less than 200 MB. |
| TC-INFRA-002 | Image runs as non-root | INFRA | Must | FR-CON-01, NFR-SEC-01 | App image has been built. | Inspect runtime user metadata. | Image built from the pull request commit | Runtime user is not root and not UID 0. |
| TC-INFRA-003 | Image contains no local env secret | INFRA | Must | FR-CON-01, NFR-SEC-03 | Local env contains fake secret before build. | Build image and inspect history or filesystem. | Fake variable `FAKE_SECRET=shipit-demo-secret` | Secret text is not found. |
| TC-INFRA-004 | Compose starts app and PostgreSQL healthy | INFRA | Must | FR-CON-02, BR-12 | Docker Engine 29.x is running. | Start Compose stack and wait for health. | Services `app` and `postgres` | Both services report healthy within the documented timeout. |
| TC-INFRA-005 | Compose fails clearly without database config | INFRA | Must | FR-CON-02, BR-09 | Ignored env file lacks required database configuration. | Run `make local-start`. | Missing DATABASE_URL | Exit 2 `secret-value-missing: DATABASE_URL` before starting services; no secret output. Direct app config validation is separately covered by TC-UT-008. |
| TC-INFRA-006 | Required checks list all mandatory gates | INFRA | Must | FR-CI-01, BR-15 | PR or push to `main` runs CI. | Read check names and saved acceptance artifacts. | App change | Required checks include Python/shell/container/security/Helm/schema/Terraform gates plus local acceptance for Compose and one-node kind; failures block publishing/merge. |
| TC-INFRA-007 | Coverage gate blocks 79 percent line coverage | INFRA | Must | FR-CI-01, NFR-MAINT-01 | Test branch reports 79% line coverage. | Run PR workflow. | Coverage report line 79%, branch 75% | Workflow fails because line coverage is below 80%. |
| TC-INFRA-008 | GHCR publish uses immutable version tags | INFRA | Must | FR-CI-02, BR-16 | Main workflow runs for app version `v1.2.0` and commit `abc1234`. | Inspect published image tags and retry the SemVer tag with a different digest. | SemVer `v1.2.0`, SHA `abc1234` | GHCR contains both tags for one digest; reusing `v1.2.0` for another digest is rejected. |
| TC-INFRA-009 | Branch protection blocks failing secret scan | INFRA | Must | FR-CI-03, BR-17 | Branch protection is enabled. | Attempt to merge PR with failing Gitleaks check. | PR status contains failed Gitleaks | GitHub blocks merge. |
| TC-INFRA-010 | Namespaces and profile replicas exist | INFRA | Must | FR-K8S-01, BR-18 | Local kind profile installed with prod active. | List namespaces, app/database pods, and nodes. | Lite then standard values | Both namespaces exist; prod has lite 1/standard 2 ready app pods and 1 database pod; inactive dev app/database 0; lite has exactly 1 node. |
| TC-INFRA-011 | Gateway routes prod host | INFRA | Must | FR-K8S-02, BR-21 | Envoy Gateway is installed and prod app is ready. | Request prod health through `shortlink.localhost`. | Prod host header | Response is 200 from prod and not from dev. |
| TC-INFRA-012 | Gateway separates sequential environments | INFRA | Must | FR-K8S-02, BR-21 | Capture prod evidence, then stop prod and activate dev. | Request each Host at loopback 8081. | Dev/prod Host headers | Dev health is 200; inactive prod has no ready endpoints and returns 503, never dev's body; namespace/database isolation remains. |
| TC-INFRA-013 | Missing secret source fails secret creation | INFRA | Must | FR-K8S-03, BR-22 | Secret source file is absent. | Run secret creation procedure. | Missing local secret file | Procedure exits non-zero with message `secret-file-missing`. |
| TC-INFRA-014 | Pod security settings are rendered | INFRA | Must | FR-K8S-04, BR-23 | Helm chart is rendered for prod. | Check rendered app container settings. | Prod values | Container has non-root, read-only root filesystem, drops ALL capabilities, and has requests and limits. |
| TC-INFRA-015 | Readiness removes pod from service | INFRA | Must | FR-K8S-04, BR-24 | One prod pod can be made not ready. | Force readiness endpoint to return 503 and list Service endpoints. | Affected pod name | Affected pod is absent from Service endpoints. |
| TC-INFRA-016 | PostgreSQL resources are separate | INFRA | Must | FR-K8S-05, BR-19 | Dev and prod are installed. | List StatefulSets, PVCs, and Services in both namespaces. | No request body. | Each namespace has one PostgreSQL StatefulSet, one PVC, and one Service; names are environment-specific. |
| TC-INFRA-017 | Helm chart passes lint and test hook | INFRA | Must | FR-HELM-01, BR-27 | Chart exists and release is installed. | Run Helm lint and Helm test. | Chart `shortlink` | Lint exits 0; Helm test verifies `/healthz` and `/readyz`. |
| TC-INFRA-018 | Environment/profile/local values render | INFRA | Must | FR-HELM-02, BR-18, BR-16 | Chart renders dev/prod plus lite/standard/local overlays. | Render active/inactive variants. | Immutable loaded local tag | Active dev 1; prod lite 1/standard 2; inactive app/database 0; correct hosts, retained PVCs, and `IfNotPresent` or `Never` (never `Always`). |
| TC-INFRA-019 | Terraform plan rejects unknown profile | INFRA | Must | FR-IAC-02, BR-29 | Terraform root is initialized. | Run Terraform plan with invalid profile. | `profile=unknown` | Plan fails before apply and names valid values `lite` and `standard`. |
| TC-INFRA-020 | Terraform outputs local platform endpoints | INFRA | Must | FR-IAC-01, FR-IAC-02 | Cached providers/charts/images prepared; offline apply completed. | Read outputs and loopback binds. | Lite one-node profile | Cluster `shipit-local`, dev/prod Host headers, Gateway 8081, Prometheus 9090, Alertmanager 9093, Mailpit 8025; API binds loopback 6445; no mandatory Grafana URL or network fetch. |
| TC-INFRA-021 | kubeconform validates rendered manifests | INFRA | Must | FR-CI-01, FR-K8S-01 | Helm renders dev and prod resources. | Run kubeconform over rendered output. | Rendered manifests | 100% of rendered manifests are valid and no resource is skipped silently. |
| TC-INFRA-022 | kubeconform uses CRD schemas | INFRA | Must | FR-CI-01 | Rendered manifests include Gateway API and Prometheus Operator resources. | Run kubeconform with schema locations for official or maintained CRD schemas. | Gateway, HTTPRoute, ServiceMonitor, PrometheusRule | Check fails if a custom kind has no matching schema or is skipped. |
| TC-INFRA-023 | Prod PDB protects voluntary disruption | INFRA | Should | FR-K8S-07, BR-56 | PDB Should item is implemented. | Drain a 1-node cluster, then repeat on a multi-node cluster under k6. | `minAvailable: 1` | 1-node drain is blocked as expected; multi-node drain completes with 0 failed requests. |
| TC-SEC-001 | Trivy blocks critical image finding | SEC | Must | FR-SEC-01, NFR-SEC-02 | Test image report contains one CRITICAL finding. | Run the image vulnerability gate. | CRITICAL OpenSSL finding | CI fails and no GHCR image is pushed. |
| TC-SEC-002 | Trivy blocks HIGH with fix | SEC | Must | FR-SEC-01, BR-36 | Test image report contains HIGH finding with fixed version. | Run the fix-available vulnerability gate. | HIGH package with fix available | CI fails and report names package and fixed version. |
| TC-SEC-003 | Gitleaks blocks committed secret pattern | SEC | Must | Secret hygiene NFR-SEC-03; scan FR-SEC-01; CI FR-CI-01 | Throw-away branch includes a synthetic, never-issued value in the GitHub personal-access-token format. | Run secret scan gate and delete the branch after evidence is saved. | Described fixture only; no real-looking token is written in this specification. | CI fails with a Gitleaks finding for the GitHub PAT rule and the branch is never merged. |
| TC-SEC-004 | Dependabot covers three ecosystems | SEC | Must | FR-SEC-02, BR-37 | Dependency automation is configured. | Review Dependabot coverage. | Ecosystems GitHub Actions, pip, Docker | All three ecosystems are present. |
| TC-OBS-001 | Prometheus API confirms active targets | E2E | Must | FR-OBS-01, NFR-OBS-01 | Only prod active with ServiceMonitor. | Query `/api/v1/targets` and `up` through loopback API. | Prod label; inactive dev | Saved success JSON has value 1 for each ready prod target; inactive dev has no discovered app target or target-down alert. |
| TC-OBS-002 | Prometheus RED/resource evidence is saved | E2E | Must | FR-OBS-02, BR-31 | Quiet local fixture window; active prod scraped. | Capture counters, generate 100 valid 302/10 expected 404 requests without following redirects, wait for scrape, execute document 08 queries and save JSON/text. | Local seed codes; recorded API parser fixture | RED request delta 110, successful-redirect delta 100, 5xx delta 0; saved API success results include measured rate, p50/p95/p99, CPU, bytes, restarts and units/labels/timestamps. Restart fixture count 1 is read as 1; no panels/exports. |
| TC-OBS-003 | High error rate alert fires | E2E | Must | FR-OBS-03, BR-32 | Prometheus has loaded the error-rate rule. | Generate 5xx rate above 5% for 5 minutes. | 6 failed requests per 100 total | `ShortLinkHighErrorRate` fires with severity critical. |
| TC-OBS-004 | 450 ms p95 does not fire latency alert | Rule test | Must | FR-OBS-03, BR-32 | Local latency rule and recorded histogram-series fixture loaded. | Evaluate for 10 minutes using bundled Prometheus rule tooling. | Fixture p95 exactly 0.450 seconds | `ShortLinkHighLatencyP95` is not firing; saved rule output distinguishes seconds from milliseconds. |
| TC-OBS-005 | Mailpit receives target down alert | E2E | Must | FR-OBS-04, BR-33 | Alertmanager routes to Mailpit. | Make ShortLink target down for 5 minutes. | Prod target unavailable | Mailpit receives at least 1 matching e-mail with environment `prod`, severity `critical`, and a runbook link; duplicates are allowed. |
| TC-OBS-006 | High latency alert fires | E2E | Must | Alert rule FR-OBS-03; coverage NFR-OBS-02; threshold BR-32 | Prometheus has loaded the latency rule and test delay is enabled. | Inject 600 ms delay for at least 10 minutes. | Prod redirect route with delay | `ShortLinkHighLatencyP95` fires with severity warning. |
| TC-OBS-007 | Crash-loop alert fires | E2E | Must | Restart alert FR-OBS-03; runbook NFR-OBS-02; threshold BR-32 | Prometheus has loaded the crash-loop rule. | Create at least 3 container restarts within 10 minutes and keep the condition for 5 minutes. | Prod app pod restart evidence | `ShortLinkPodCrashLooping` fires with severity critical. |
| TC-REL-001 | Rolling update has zero failed requests | PERF | Must | FR-K8S-06, FR-REL-02, NFR-REL-01 | Only prod active; lite 1/standard 2 ready replicas; distinct good/new/rollback images loaded; surge reserve available. | Run k6 without redirect following; upgrade to loaded local tag with maxUnavailable 0/maxSurge 1; save rollout/k6 output. | Lite 20 VUs/1 replica; standard 50 VUs/2 replicas | 0 unexpected/failed requests in both profiles; rollout finishes without pulls; transient max app pods lite 2/standard 3 and observed memory recorded. |
| TC-REL-002 | Pod kill recovers within 120 seconds | E2E | Must | FR-REL-02, NFR-REL-02 | Active prod has lite 1/standard 2 app pods. | Delete one pod and watch replacement via kubectl. | Selected pod | Replacement Ready within 120 seconds; lite traffic interruption is recorded, not misclaimed as zero downtime. |
| TC-REL-003 | Offline Helm rollback restores release | E2E | Must | FR-REL-02, BR-35 | Bad update revision exists and previous good local image is loaded. | With external egress denied, roll back and smoke-test. | Prior revision and loaded local tag | Smoke exits 0, Helm history names restored revision, and saved metrics/alerts show recovery without registry pulls. |
| TC-PERF-001 | Redirect p95 reference and lite evidence | PERF | Must | NFR-PERF-01, FR-APP-02 | Only prod active at profile count. | Run same k6 scenario, no redirect following/cache. | Standard 50 VUs/2 replicas; lite 20 VUs/1 replica; 302 and expected 404 separate | Standard valid-redirect p95 below 300 ms; lite measurements saved; unexpected/transport errors below 1%; no external URL requests. |
| TC-PERF-002 | Create p95 reference and lite evidence | PERF | Must | NFR-PERF-02, FR-APP-01 | Only prod active at profile count. | Run create-link k6 scenario without target URL fetches. | 20 VUs; standard 2/lite 1 replicas; unique fictional URLs | Standard create p95 below 500 ms for 5 minutes; lite results saved with profile/counts; trainer may verify reference target separately. |
| TC-PERF-003 | Standard HPA scales above two replicas | PERF | Should | FR-SCALE-01, NFR-SCALE-01 | Standard-only HPA and separate memory/surge pre-check complete. | Run CPU load and inspect kubectl/metric evidence. | CPU above 70%; min 2/max 6 | Replicas increase above 2 and return to 2 within bound 6; no lite HPA/multi-node requirement. |
| TC-SEC-005 | NetworkPolicy blocks debug pod | SEC | Should | FR-NET-01, NFR-NET-01 | Calico or Cilium and NetworkPolicy are enabled. | Connect to PostgreSQL from app and from unlabeled debug pod. | PostgreSQL service address | App connection succeeds; debug pod connection is denied. |
| TC-E2E-001 | Loki API finds request ID | E2E | Should | FR-LOG-01, NFR-LOG-01 | Separately budgeted standard Loki/Alloy enabled. | Send request and query Loki HTTP API/CLI. | `req-loki-001` | Saved log result contains the request ID within 5 minutes; no visualization implementation. |
| TC-INFRA-024 | Jenkins stages mirror GitHub Actions | INFRA | Should | FR-JENKINS-01, BR-49 | Jenkins explanation exists. | Compare Jenkins stage list with CI gate list. | Gate list from BR-15 | Each GitHub Actions gate maps to one Jenkins stage. |
| TC-DOC-001 | Alert runbooks contain required sections | DOC | Must | FR-REL-01, NFR-OBS-02 | Four runbook files exist. | Review each runbook heading list. | Required sections from document 08 | All four runbooks contain symptoms, impact, diagnosis, mitigation, escalation, and evidence sections. |
| TC-DOC-002 | AWS mapping table covers required rows | DOC | Must | FR-DOC-01, BR-38 | Local-to-AWS table exists. | Check each required local platform item. | kind, GHCR, PVC, Gateway, Alertmanager e-mail, local Terraform state | Every required row has one AWS equivalent and one interview point. |
| TC-CLI-009 | Secret creation reports missing file | CLI | Must | FR-K8S-03, BR-22 | `scripts/create-secrets.sh` exists. | Run create-secrets with a missing file. | `--env prod --file missing.env` | Exit code is 2 and output includes `secret-file-missing`. |
| TC-INFRA-025 | Offline artifact inventory detects drift | INFRA | Could | FR-LOCAL-02, BR-54 | Cached charts and old/new/rollback images have a local digest inventory. | Verify inventory without egress; then change one chart fixture byte and verify again. | Prepared local artifact set | Valid set exits 0; changed set exits 2 `artifact-digest-mismatch` naming only the artifact; no registry query or secret output. |

## Local acceptance catalog

Run `make local-test STAGE=compose` then, after stopping Compose, `make local-test STAGE=kind PROFILE=lite` (commands students implement as the acceptance runner). Each runs TC-LOCAL-001 through TC-LOCAL-004 in an isolated project-local workspace with external egress denied but loopback/container/cluster traffic allowed. Download/cache/image/cert preparation and obtaining the clean clone happen before isolation. No hosted registry or cloud subscription is required. Exercise dev/prod kind sequentially, never together on lite.

| ID | Linked IDs | Preconditions and operation | Exact expected result |
|---|---|---|---|
| TC-LOCAL-001 | Startup/migrations: FR-LOCAL-01, FR-CON-02, FR-K8S-01, FR-K8S-05, NFR-PORT-01, NFR-DOC-01, BR-12, BR-57 | Clean clone with caches/images and ignored secret files prepared; no stage data. Run stage start, migrations/seed, health/ready, and local demo. | Compose prints `local-ready stage=compose env=dev`; kind prints matching stage/env with lite 1 app/1 database pod, inactive workloads 0, and 1 node. Health status ok, readiness ready; schema current; exactly 1 seed row; local app tag loaded with permitted pull policy. |
| TC-LOCAL-002 | Offline API/metrics: FR-LOCAL-01, FR-APP-01, FR-APP-02, FR-OBS-02, BR-01, BR-05, BR-07, BR-57 | Deny external egress after preparation. Run stage demo twice, then POST fictional valid URL, without following redirects. | Each demo prints document 06 summary: health/ready 200; seed 302 with exact Location; missing 404 `short-code-not-found`. Successful redirect series increases 1 per demo; seed count stays 1. POST returns 201/7-character Base62 code without fetching target. kind saves success metric/alert API evidence; no pulls/downloads. |
| TC-LOCAL-003 | FR-LOCAL-01, FR-K8S-05, FR-BASH-01, NFR-REL-03, BR-19, BR-57 | Store a new fictional link, record code/id and volume/PVC identity. Stop/start same stage/env; try reset without confirmation. | Stop prints matching `local-stopped stage=... data-preserved`; original code/id and storage identity remain; seed is not duplicated. Reset exits 2 `local-reset-confirmation-required` and changes no data. Separate dev/prod test markers do not cross databases when switching sequentially. |
| TC-LOCAL-004 | FR-LOCAL-01, FR-APP-03, FR-APP-04, FR-K8S-03, NFR-DOC-01, BR-02, BR-09, BR-22, BR-57 | Isolated cases: invalid URL/profile; unavailable Docker; absent secret/cache; occupied stage port; stop active PostgreSQL after app ready. | Invalid URL 422 `target-url-invalid`, no row; invalid profile exits 2 `profile-invalid`; prerequisites exit 2 `local-dependency-unavailable: docker`, `secret-file-missing`, `local-artifact-missing: <path>`, or `local-port-in-use: <port>`. Direct app health remains 200; direct readiness is 503/status not_ready/error database-not-ready (use in-pod request in kind after endpoints removed). Startup health deadline exits 3 `local-readiness-timeout: postgres`. No erase/download/registry fallback. |

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
| FR-LOCAL-01 | TC-LOCAL-001, TC-LOCAL-002, TC-LOCAL-003, TC-LOCAL-004 (both stages) |
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
| FR-SLO-01 | Demo step: saved SLO query/rule JSON names 99.5%/300 ms design targets; demonstrate burn alert with recorded series. |
| FR-POST-01 | Demo step: review bad-release postmortem and DORA notes. |
| FR-HELM-03 | Demo step: run helm-unittest for dev host, prod replicas, and security context values. |
| FR-IAC-03 | Demo step: run Terraform test and TFLint evidence. |
| FR-JENKINS-01 | TC-INFRA-024 |
| FR-ANSIBLE-01 | Demo step: Multipass Ubuntu VM reaches kind version. |
| FR-ROLLOUT-01 | Demo step: canary receives 10% traffic before promotion. |
| FR-SIGN-01 | Demo step: unsigned image is rejected in a test namespace. |
| FR-OTEL-01 | Demo step: show one trace from API handler to database call. |
| FR-LOCAL-02 | TC-INFRA-025 |
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
| NFR-PORT-01 | TC-LOCAL-001 to TC-LOCAL-004 plus recorded Windows WSL2/macOS/Linux entrypoint checks. |
| NFR-HW-01 | Trainer pre-check: one-node lite, one active environment, proposed versus observed startup/steady/load/surge caps; TC-LOCAL-001 to TC-LOCAL-004. |
| NFR-HW-02 | TC-CLI-001 |
| NFR-CI-01 | CI-enforcement evidence: TC-INFRA-006, TC-INFRA-007, TC-INFRA-009, TC-INFRA-022 |
| NFR-DOC-01 | TC-LOCAL-001 to TC-LOCAL-004 and trainer clean-clone guide in 10 steps or fewer. |
| NFR-LIC-01 | Demo step: ADR and licence notes for Terraform, OpenTofu, Valkey, Bitnami, and Mailpit choices. |
| NFR-PRIV-01 | TC-UT-003 and data review using only fictional domains. |
| NFR-SCALE-01 | TC-PERF-003 |
| NFR-NET-01 | TC-SEC-005 |
| NFR-LOG-01 | TC-E2E-001 |
| NFR-SLO-01 | Saved Prometheus SLO query/rule and burn-alert evidence; no 30-day measurement claim without history. |

## Business rule evidence matrix

| Business rule IDs | Evidence |
|---|---|
| BR-01 | Code-generation evidence: TC-UT-005, TC-UT-006, TC-IT-001 |
| BR-02 | TC-UT-001, TC-UT-002, TC-UT-003, TC-UT-004, TC-IT-002 |
| BR-03 | TC-UT-007, TC-API-002 |
| BR-04 | Lookup-state evidence: TC-IT-004, TC-IT-005, TC-IT-006 |
| BR-05 | TC-IT-003 |
| BR-06 | TC-IT-008 and saved successful/failed redirect-series query review. |
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
| BR-54 | TC-INFRA-025 |
| BR-55 | FR-DBOP-01 demo step. |
| BR-56 | TC-INFRA-023 |
| BR-57 | TC-LOCAL-001, TC-LOCAL-002, TC-LOCAL-003, TC-LOCAL-004 |

## Performance-test conditions

| Item | Required condition |
|---|---|
| Hardware | Standard 16 GB profile for the 50-VU acceptance run; 8 GB lite profile records the same script at 20 VUs. |
| Cluster | kind 0.33.x/Kubernetes 1.37.x; only prod active: standard 2 ready app replicas, lite 1 on exactly 1 node; inactive workloads 0. |
| Data volume | At least 1,000 active links, 1 expired link, 1 deleted link, and 100 missing-code samples. |
| Warm-up | 2 minutes of low-rate traffic before measured run. |
| Redirect run | 50 virtual users for 5 minutes on standard, or 20 virtual users on lite evidence; redirect following disabled; valid redirects and expected misses are measured separately. |
| Create run | 20 virtual users for 5 minutes; each request uses a unique `https://example.in/perf/{n}` URL. |
| Thresholds | Standard reference: valid redirect p95 below 300 ms/create p95 below 500 ms. Lite records 20-VU/1-replica values; trainer may verify reference separately. Both: unexpected/transport errors below 1%, rolling update 0 failed requests. |
| Evidence | Save k6 summary, Prometheus query/alert JSON and timestamps, Git SHA, loaded image tag/pull policy, profile, node/replica counts, and observed/proposed memory including surge. |

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
| Evidence | Redacted logs, command output, saved metric/alert JSON, k6 summary, or separate connected CI URL. |
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
- Standard reference k6 proves redirect p95 below 300 ms; lite saves 20-VU/1-replica measurements. Both profiles prove rolling update 0 failed requests with surge 1.
- Reliability evidence includes rolling update, rollback, pod kill, and backup restore.
- Every Must FR, BR, and NFR has a linked test case or named demo step.
- TC-LOCAL-001 through TC-LOCAL-004 pass for both stages with saved offline/persistence/error evidence; at least 45 meaningful cases remain. No frontend/cloud deliverable is assessed.

[Back to README](../README.md)
