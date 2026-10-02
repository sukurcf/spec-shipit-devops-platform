# DevOps, CI/CD, and quality

Purpose: This document defines ShipIt engineering workflow, CI/CD gates, container rules, configuration, release practice, and Definition of Done.

## Git workflow

ShipIt uses GitHub Flow. `main` is always releasable. Every change starts from an issue and reaches `main` through a pull request.

| Rule | Requirement |
|---|---|
| Branch names | Use `feature/FR-APP-01-create-link`, `fix/FR-CI-01-coverage-gate`, `docs/FR-DOC-01-aws-mapping`, or `chore/FR-SEC-02-dependabot`. |
| Pull request size | Keep PRs small enough to review in 20 minutes. One PR SHOULD cover one feature slice or one platform gate. |
| Reviews | The trainer reviews at least 2 substantive PRs each week. The student MAY merge only after CI is green and the checklist is complete. |
| Direct push | Direct push to `main` MUST be disabled. |
| Issues | Each meaningful task links to an issue or a GitHub Projects card. |

## Conventional Commits

| Type | ShipIt example | Use when |
|---|---|---|
| `feat` | feat(app): add POST /api/v1/links for FR-APP-01 | A new requirement works. |
| `fix` | fix(helm): keep prod replicas during readiness failure | A bug is corrected. |
| `test` | test(ci): add coverage failure case for FR-CI-01 | Tests or evidence improve. |
| `docs` | docs(runbook): add latency alert steps for FR-REL-01 | Documentation changes only. |
| `chore` | chore(deps): update Trivy version for FR-SEC-02 | Tooling or dependency maintenance. |
| `refactor` | refactor(app): isolate redirect metrics recording | Behavior stays the same. |

## Pull request checklist

- The PR links the related `US-NN`, `FR-AREA-NN`, `BR-NN`, or `NFR-CAT-NN` IDs.
- CI gates pass on the latest commit.
- New or changed behavior has tests or named demo evidence.
- Coverage stays at line at least 80% and branch at least 70%.
- Docker images stay below 200 MB and run as non-root.
- Kubernetes resources render and pass kubeconform with required CRD schemas.
- Terraform format and validation are green when platform files change.
- No secret appears in commits, image layers, logs, screenshots, or runbooks.
- The PR description names significant AI help when used.
- The student can explain the change without reading generated text.
- TC-LOCAL-001 through TC-LOCAL-004 pass for Compose and one-node kind after prepared downloads, with external egress denied and saved seed/persistence/failure evidence.
- Changed image/chart values still permit loaded local tags and enforce one active environment, correct profile replicas, surge reserve, and loopback bindings.

## Branch protection

| Setting | Required value |
|---|---|
| Required PR | Enabled before merge to `main`. |
| Required checks | Ruff, mypy, pytest coverage, hadolint, ShellCheck, image build, Trivy, Gitleaks, Helm lint, kubeconform, Terraform format/validate, local acceptance (Compose and one-node kind). |
| Stale approvals | Dismiss approval after a new commit. |
| Direct push | Blocked for everyone. |
| Force push | Blocked. |
| Required conversation resolution | Enabled. |

## Code quality tools and rules

| Area | Tool | ShipIt rule |
|---|---|---|
| Python format and lint | Ruff 0.16.x | No lint errors. Formatting runs before tests. |
| Python types | mypy 2.4.x | Application packages require typed functions. Tests and migrations may be excluded with a note. |
| Python tests | pytest 9.1.x and coverage 7.16.x | Line coverage at least 80%; branch coverage at least 70%. |
| Dockerfile | hadolint 2.15.x | Multi-stage build, slim base, no secrets, and non-root runtime. |
| Bash | ShellCheck 0.11.x | `bootstrap.sh`, `smoke-test.sh`, `db-backup.sh`, and `db-restore.sh` have 0 findings. |
| Secrets | Gitleaks 8.30.x | Any real secret finding blocks merge. |
| Image security | Trivy 0.75.x | CRITICAL fails. HIGH with a fix fails. Accepted HIGH misconfiguration needs written justification. |
| Helm | Helm lint | Chart name is `shortlink`; dev and prod values render. |
| Kubernetes schemas | kubeconform 0.8.x | 100% of rendered Kubernetes and custom resources validate. Gateway API and Prometheus Operator resources need version-matched CRD schemas. Missing schemas and skipped resources fail. |
| Terraform | Terraform 1.16.x | Format check and validate are required. TFLint is Should. |

## CI pipeline table

| Job | Trigger | Steps in words | Gate or failure condition |
|---|---|---|---|
| Repository hygiene | Pull request and push to `main` | Check committed files, lockfile, line endings, and forbidden secret files. | Fails if a required lockfile is missing or a local secret source file is committed. |
| Python quality | Pull request and push to `main` | Install with uv, run Ruff, run mypy, and run pytest with coverage. | Fails on lint, type errors, test failure, line coverage below 80%, or branch coverage below 70%. |
| Container quality | Pull request and push to `main` | Build the ShortLink image, inspect size and user, run hadolint, and verify health-check presence. | Fails if image is 200 MB or larger, runtime user is root, or Dockerfile lint fails. |
| Script quality | Pull request and push to `main` | Run ShellCheck on the required scripts and test their help output. | Fails on any ShellCheck finding or missing help text. bats runs only when the Should bats item is implemented. |
| Security scan | Pull request and push to `main` | Run Gitleaks and Trivy image plus config scans. | Fails on secrets, CRITICAL image vulnerabilities, HIGH vulnerabilities with fixes, or unexplained HIGH misconfigurations. |
| Helm validation | Pull request and push to `main` | Run Helm lint, render dev and prod values, and verify the `helm test` hook exists. | Fails if render breaks, chart metadata is invalid, or required resources are absent. |
| Kubernetes schema | Pull request and push to `main` | Validate rendered Kubernetes, Gateway API, and Prometheus Operator resources with kubeconform and CRD schemas. | Fails when any resource is invalid, skipped, or missing a schema. |
| Terraform validation | Pull request and push to `main` | Run Terraform format check and validate platform configuration. | Fails on formatting drift, invalid variables, or missing provider configuration. |
| Local acceptance | Pull request and push to `main` | On an Ubuntu runner, prepare local dependencies/images/provider/chart/schema caches and ignored test secrets. Deny external egress while allowing loopback/container/cluster traffic; run `make local-test STAGE=compose`, stop Compose, then `make local-test STAGE=kind PROFILE=lite`. Save TC-LOCAL-001 to TC-LOCAL-004 output, local image/tag/pull-policy evidence, data IDs and metrics/alert JSON. | Mandatory failure on seed/demo/health/persistence/error mismatch, attempted external download/pull, or incompatible local profile. Both environments are exercised sequentially, not a two-environment memory claim. |
| Main image publish | Push to `main` after checks | Publish GHCR image tags for the declared SemVer app version and short SHA. | Fails if required checks are not green, image push is denied, or the SemVer tag already points to another digest. |
| Optional release | Tag `vX.Y.Z` | Reuse gates, publish release notes, and record image tags. | Fails if tag is not SemVer or security gates fail. |

Hosted runner provisioning, cache preparation, vulnerability scans, and GHCR publication need internet. The isolated runtime acceptance phase does not. CI builds/loads the local image without depending on a previously published GHCR image. WSL2/macOS/Linux trainer pre-check supplements Linux CI. There is no frontend coverage, dashboard export, or cloud deployment job.

## Docker and Compose requirements

| Requirement | Exact ShipIt value |
|---|---|
| Image style | Multi-stage build with a slim Python runtime base. |
| Runtime user | Non-root user. |
| Image size | Below 200 MB. |
| Health check | Uses `/healthz` and expects 200. |
| Secrets | No secrets in build arguments, layers, image history, or logs. |
| Compose services | `app` and `postgres` are Must. Optional cache is Should. |
| Compose health | PostgreSQL health precedes one migration/seed step and app readiness; missing credentials fail clearly before start. |
| Tags | Base and service images use pinned version tags, never floating tags. |
| Volumes | `shipit-compose-postgres` persists across stop/start; kind retains separate PVCs/local bind mounts. Confirmed reset only; backups retained. |
| Local limits/ports | Follow proposed document 06 caps; Compose app loopback 8080, kind Gateway loopback 8081. No host PostgreSQL port; never run Compose and kind or both app environments concurrently in the canonical profile. |
| Image delivery | Local `shortlink:local-<short-sha>` from cached inputs; `kind load docker-image shortlink:local-<short-sha> --name shipit-local` before Helm. `IfNotPresent` or `Never` permits loaded tags. Connected GHCR delivery separately pins SHA/digest. |

## Environment variables

| Name | Example value | Purpose | Secret? |
|---|---|---|---|
| APP_ENV | dev | Selects app environment and log field. | No |
| DATABASE_URL | postgresql URL for local ShortLink dev database | Connects to PostgreSQL. | Yes |
| BASE_URL | Compose `http://127.0.0.1:8080`; kind prod `http://shortlink.localhost:8081` | Builds `short_url`; kind smoke uses loopback 8081 and explicit Host header. | No |
| LOG_LEVEL | INFO | Controls JSON log verbosity. | No |
| API_KEY | local placeholder secret | Protects optional delete endpoint. | Yes |
| PROMETHEUS_MULTIPROC_DIR | metrics state directory | Supports metrics if multiple workers are used. | No |
| CACHE_URL | local Valkey URL | Optional cache-aside redirect cache. | No for local; yes if authenticated |
| CACHE_TTL_SECONDS | 60 | Optional redirect cache TTL. | No |
| POSTGRES_DB | shortlink_dev | Local PostgreSQL database name. | No |
| POSTGRES_USER | shortlink | Local PostgreSQL user. | No |
| POSTGRES_PASSWORD | local placeholder password | Local PostgreSQL password. | Yes |

The student repository SHOULD include an `.env.example` with names and safe placeholders only. It MUST NOT include real secrets.

Local start reads ignored `.local/compose-secrets.env`, `.local/dev-secrets.env`, or `.local/prod-secrets.env` after explicit preparation. Test credentials are generated locally, not injected from GitHub for startup. Stop preserves state and only stops owned services/port-forward processes; reset needs `CONFIRM=DELETE_LOCAL_DATA`. No paid account, public hostname, API key, or cloud resource is required.

## Versioning and release

| Item | Rule |
|---|---|
| Application version | Use SemVer, such as `v1.2.0`, from the declared app version. |
| Image tags | Publish both immutable SemVer and short SHA, such as `v1.2.0` and `abc1234`. Reusing SemVer for a different digest is rejected. |
| Prod deployment pin | Local lab: immutable loaded `local-<short-sha>` tag; CI registry-delivery exercise: short SHA/digest. No floating `latest` or mandatory local registry login. |
| Helm chart version | Increment when templates or values change. |
| Release evidence | Record image tag, chart version, target environment, smoke result, and rollback note. |
| CHANGELOG | Student repository keeps a human-readable changelog for releases. |
| Rollback | Prod rollback uses Helm release history and records the revision in evidence. |

## Dependency management

| Ecosystem | Requirement |
|---|---|
| Python | Manage with uv and commit `uv.lock`. Dependabot covers pip dependencies. |
| GitHub Actions | Dependabot opens PRs for action version updates. |
| Docker | Dependabot monitors Docker base images. |
| Kubernetes add-ons | Version changes are reviewed through PRs and ADRs when behavior changes. |
| Terraform providers | Provider versions are pinned and updated through reviewed PRs. |
| Vulnerabilities | Security updates use the same gates as feature work. |

## Definition of Done

A ShipIt item is Done only when all applicable points are true.

- The linked requirement behavior works locally.
- Tests or documented demo evidence cover the behavior.
- Required CI gates are green.
- The app image stays below 200 MB and runs as non-root when container work changes.
- Kubernetes changes render for dev and prod.
- Terraform changes validate and explain state impact.
- Documentation and runbooks mention new commands, alerts, or evidence.
- No secret is committed.
- The PR is small, reviewed, and merged through branch protection.
- The student can answer why the change exists and how it fails safely.
- Start/stop/demo/reset meet document 06, all four local cases pass for both stages, and Prometheus/Alertmanager API evidence replaces all custom visualization work.

[Back to README](../README.md)
