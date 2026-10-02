# ShipIt DevOps platform

ShipIt is a six-week DevOps project where one fresher builds a local cloud-native delivery platform for the ShortLink URL shortener.

## Quick facts

| Item | Value |
|---|---|
| Track | DevOps / Cloud Native |
| Difficulty | ★★★★☆ Upper-intermediate |
| Duration | 6 weeks, about 180 hours |
| Target job roles | DevOps Engineer, Cloud Engineer, Site Reliability Engineer, Platform Engineer, Build and Release Engineer |
| Key skills | Linux, Bash, Docker, GitHub Actions, Kubernetes, Helm, Terraform, Prometheus, Grafana, alerting, runbooks |
| Prerequisites | Basic Python, basic SQL, Git basics, HTTP basics |
| Minimum hardware | 8 GB RAM with the lite profile |
| Recommended hardware | 16 GB RAM with the standard profile |
| Student repository name | `shipit` |

## What you will build

- A FastAPI ShortLink service with `POST /api/v1/links`, `GET /{code}`, `/healthz`, `/readyz`, and `/metrics`.
- A secure Docker image below 200 MB and a Docker Compose development stack.
- Operational Bash scripts for bootstrap, smoke tests, PostgreSQL backup, and restore.
- GitHub Actions PR gates for lint, tests, coverage, scans, Helm, Kubernetes schemas, and Terraform.
- A kind 0.33.x cluster with `shortlink-dev` and `shortlink-prod` namespaces.
- A Helm 4.x chart named `shortlink` with dev and prod values.
- Terraform 1.16.x that creates kind and installs Envoy Gateway and kube-prometheus-stack.
- Grafana dashboards, Prometheus alerts, Mailpit alert e-mail, and alert runbooks.
- Reliability evidence for rolling update, rollback, and pod self-healing drills.
- Interview notes that map kind to EKS, GHCR to ECR, PVC to EBS, Gateway to a load balancer, alert e-mail to SNS or SES, and local Terraform state to S3 remote state.

## Architecture at a glance

```mermaid
flowchart TD
  Dev["Application developer"] --> PR["GitHub pull request"]
  PR --> CI["GitHub Actions gates"]
  CI --> GHCR["GHCR image tags"]
  GHCR --> Helm["Helm shortlink chart"]
  Terraform["Terraform platform apply"] --> Kind["kind Kubernetes cluster"]
  Terraform --> Envoy["Envoy Gateway"]
  Terraform --> Mon["kube-prometheus-stack"]
  Helm --> DevNs["shortlink-dev namespace"]
  Helm --> ProdNs["shortlink-prod namespace"]
  Envoy --> App["ShortLink FastAPI pods"]
  App --> PG["PostgreSQL StatefulSet and PVC"]
  Mon --> Grafana["Grafana RED dashboard"]
  Mon --> Alertmanager["Alertmanager"]
  Alertmanager --> Mailpit["Mailpit e-mail"]
  SRE["On-call SRE"] --> Grafana
  SRE --> Mailpit
```

## How to read this specification

1. Read the overview, users, functional requirements, and non-functional requirements first.
2. Read the architecture, data model, and platform specification before you create implementation folders.
3. Read setup, DevOps, milestones, testing, rubric, interview, and glossary documents before the first Friday demo.
4. `MUST` means mandatory for grading. `SHOULD` means recommended after Must scope. `MAY` means optional stretch work.
5. IDs keep work traceable: user stories use `US-NN`, functional requirements use `FR-AREA-NN`, business rules use `BR-NN`, non-functional requirements use `NFR-CAT-NN`, and test cases use `TC-TYPE-NNN`.

## Document index

| Document | Purpose |
|---|---|
| [01 Project overview](docs/01-project-overview.md) | Explains the problem, vision, scope, constraints, and success criteria. |
| [02 Users and roles](docs/02-users-and-roles.md) | Defines roles, personas, permissions, journeys, and user stories. |
| [03 Functional requirements](docs/03-functional-requirements.md) | Defines FRs, business rules, state machines, validation, and errors. |
| [04 Non-functional requirements](docs/04-non-functional-requirements.md) | Defines measurable quality, security, reliability, and portability targets. |
| [05 System architecture](docs/05-system-architecture.md) | Shows architecture diagrams, deployment view, repository tree, and ADR list. |
| [06 Tech stack and setup](docs/06-tech-stack-and-setup.md) | Pins versions and explains local setup for Windows, macOS, and Linux. |
| [07 Data model](docs/07-data-model.md) | Defines application data, operational evidence records, and retention. |
| [08 Platform specification](docs/08-platform-specification.md) | Defines platform environments, Kubernetes resources, IaC modules, and alert rules. |
| [09 Testing strategy and test cases](docs/09-testing-strategy-and-test-cases.md) | Defines testing strategy, test catalog, traceability, and performance conditions. |
| [10 DevOps, CI/CD, and quality](docs/10-devops-ci-cd-and-quality.md) | Defines Git workflow, CI gates, Docker rules, configuration, releases, and DoD. |
| [11 Milestones and deliverables](docs/11-milestones-and-deliverables.md) | Gives the six-week plan, effort budget, Gantt chart, submission flow, and demo script. |
| [12 Evaluation rubric](docs/12-evaluation-rubric.md) | Defines mandatory gates, scoring criteria, bonus rules, deductions, and viva questions. |
| [13 Interview preparation](docs/13-interview-preparation.md) | Provides STAR pitch, interview questions, AWS basics, resume bullets, and mock checklist. |
| [14 Glossary and resources](docs/14-glossary-and-resources.md) | Defines terms and links official documentation and free learning resources. |

## Rules for students

### Individual work

You work alone in your own public GitHub repository. You may discuss concepts with classmates, but you must write and understand your own implementation.

### AI assistant policy

> You MAY use AI assistants (for example GitHub Copilot or ChatGPT) to learn concepts, explain errors, and review your code. You MUST understand every line that you commit. You MUST tell the trainer about significant AI help in the pull request description. You MUST NOT give this specification to an AI tool and submit the generated solution as your own work. In the viva, the trainer asks you to explain and change your code live. If you cannot explain your code, the result is "Rework required".

### How to ask questions

> Open a GitHub Issue in this specification repository. Start the title with `[Question]`. The trainer answers in the issue, so all students can see the answer. If the answer changes the specification, the trainer updates the change log.

### Student repository

> Create a public repository with the name `shipit` in your own GitHub account. Add the trainer (`@sukurcf`) as a collaborator. Do not copy this specification into your repository. Link to it from your README.

### Review model

> Every change goes through a pull request. You MAY merge your own pull request after CI is green and you complete the PR checklist. The trainer reviews at least 2 substantive pull requests from each student every week. The trainer can ask for changes at any time.

### Requirement freeze

> This specification is frozen for the cohort. The trainer can add clarifications. If a change affects grading, the change log states the impact.

### Weekly demo

Every Friday, show a 15-minute demo to the trainer. Show working evidence, not slides only.

## Change log

| Version | Date | Change |
|---|---|---|
| 1.0 | 2026-10-02 | First release |
