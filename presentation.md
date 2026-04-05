# Introduction to CI/CD

**Continuous Integration & Continuous Delivery/Deployment**

*Automating the path from a developer's editor to production systems*

```
Code -> Commit -> Build -> Test -> Package -> Deploy
```

Build  |  Test  |  Deploy  |  Repeat

---

## Table of Contents

1. [Topics](#slide-01--topics)
2. [The Problem: Manual Release Hell](#slide-02--the-problem-manual-release-hell)
3. [Continuous Integration (CI)](#slide-03--continuous-integration-ci)
4. [Continuous Delivery vs Continuous Deployment](#slide-04--continuous-delivery-vs-continuous-deployment)
5. [The CI/CD Pipeline](#slide-05--the-cicd-pipeline)
6. [Stage: Build](#slide-06--stage-build)
7. [The Test Pyramid](#slide-07--the-test-pyramid)
8. [Quality Gates](#slide-08--quality-gates)
9. [Branching Strategies & Triggers](#slide-09--branching-strategies--triggers)
10. [Artefacts & Environments](#slide-10--artefacts--environments)
11. [Docker in CI/CD](#slide-11--docker-in-cicd)
12. [Tool Deep Dive: GitHub Actions](#slide-12--tool-deep-dive-github-actions)
13. [CI/CD Tool Landscape](#slide-13--cicd-tool-landscape)
14. [Secrets & Pipeline Security](#slide-14--secrets--pipeline-security)
15. [Deployment Strategies](#slide-15--deployment-strategies)
16. [Observability & Rollback](#slide-16--observability--rollback)
17. [Keeping Pipelines Fast](#slide-17--keeping-pipelines-fast)
18. [Measuring CI/CD Effectiveness: DORA Metrics](#slide-18--measuring-cicd-effectiveness-dora-metrics)
19. [Summary & Next Steps](#slide-19--summary--next-steps)

---

## Slide 01 — Topics

### Foundations

- The problem CI/CD solves
- Continuous Integration
- Continuous Delivery vs Deployment
- The pipeline concept

### Pipeline Mechanics

- Stages: build, test, package, deploy
- Triggers and branching strategies
- Artefacts and environments

### Testing in CI

- Test pyramid
- Unit, integration, E2E tests
- Code coverage and quality gates

### Tools & Practice

- GitHub Actions, GitLab CI, Jenkins
- Docker in CI/CD
- Secrets and security
- Observability and rollbacks

---

## Slide 02 — The Problem: Manual Release Hell

Before CI/CD, teams faced **integration hell** — developers worked in isolation for weeks, then merged everything at once.

### Symptoms of the old way

- Merge conflicts across thousands of lines
- "Works on my machine" — environment drift
- Manual, error-prone deployment scripts
- Releases done at midnight to reduce blast radius
- No confidence in what is actually running in prod

### Root causes

- Long-lived feature branches diverging from main
- Tests run manually, infrequently, or not at all
- Build and deploy steps lived in someone's head
- No repeatable, auditable process

### Key insight

The longer code sits unintegrated, the more expensive integration becomes. CI/CD makes integration a *continuous, cheap activity* rather than a periodic expensive one.

---

## Slide 03 — Continuous Integration (CI)

CI is a **practice** where developers integrate their changes into a shared branch frequently — ideally multiple times per day — with each integration verified by an automated build and test run.

### Core rules of CI

- Maintain a single source repository
- Automate the build
- Make the build self-testing
- Every commit triggers the pipeline
- Fix broken builds immediately — it blocks the team
- Keep the build fast (< 10 min is the target)

### CI Flow

```
Developer pushes commit
        |
        v
Pipeline triggered automatically
        |
        v
Code compiled / linted
        |
        v
Tests executed
        |
        v
Pass -> merge allowed  |  Fail -> developer notified
```

---

## Slide 04 — Continuous Delivery vs Continuous Deployment

The two CDs are often confused. The distinction is whether the final push to production is **manual** or **automatic**.

| Aspect | Continuous Delivery | Continuous Deployment |
|--------|--------------------|-----------------------|
| Definition | Code is always in a *releasable state*; deploy to prod requires a human click | Every commit that passes the pipeline is deployed to prod *automatically* |
| Human gate | **Yes** — explicit approval step | **No** — fully automated |
| Risk tolerance | Suitable when compliance, QA, or product sign-off is required | Requires very high test coverage and observability confidence |
| Typical users | Regulated industries, enterprise products | SaaS, high-cadence web services (e.g. Netflix, Etsy) |
| Prerequisite | Both require solid CI — you cannot have CD without CI | Both require solid CI — you cannot have CD without CI |

---

## Slide 05 — The CI/CD Pipeline

A **pipeline** is a sequence of automated stages. Each stage must pass before the next runs. A failure stops the pipeline and notifies the team.

```
Source       ->  Build        ->  Test             ->  Analyse        ->  Package        ->  Staging        ->  Prod
(push / PR)     (compile,        (unit,               (SAST,            (Docker,           (E2E,              (deploy
                 lint)            integration)          coverage)         wheel)             smoke)             / gate)
```

### Key properties

- **Fast feedback** — failures caught within minutes
- **Deterministic** — same inputs -> same outputs
- **Auditable** — every run logged, artefacts versioned

### Pipeline as code

Pipeline definitions live in the repo (e.g. `.github/workflows/ci.yml`). They are versioned, reviewed, and evolved alongside the application code.

---

## Slide 06 — Stage: Build

The build stage converts source code into a **runnable artefact** — a compiled binary, a Docker image, a wheel package, or a bundled web app.

### Typical tasks

- Dependency resolution (`pip install`, `npm ci`, `go mod download`)
- Compilation / transpilation
- Linting and static analysis (fail fast)
- Generating build artefact(s) with a deterministic version tag

### Use `npm ci`, not `npm install`

In CI, `ci` is stricter: it uses the lockfile exactly and fails if it would need updating — preventing silent dependency drift.

### Example: GitHub Actions build step

```yaml
# GitHub Actions build step
- name: Install deps
  run: npm ci

- name: Lint
  run: npm run lint

- name: Build
  run: npm run build

- name: Upload artefact
  uses: actions/upload-artifact@v4
  with:
    name: dist-${{ github.sha }}
    path: dist/
```

---

## Slide 07 — The Test Pyramid

Mike Cohn's **test pyramid** guides how to balance test types. More tests at the base (fast, cheap); fewer at the apex (slow, expensive).

```
        +-----------------------+
        |    E2E / UI Tests     |   few - slow - brittle - high confidence
        +-----------------------+
      +-----------------------------+
      |     Integration Tests       |   some - moderate speed - test interactions
      +-----------------------------+
  +-------------------------------------+
  |           Unit Tests                |   many - fast - isolated - milliseconds each
  +-------------------------------------+
```

### Unit tests

Test a single function or class in isolation. No I/O, no network. Should run in < 1 s total for a module.

### Integration tests

Test how components work together — e.g. service + database, or two microservices. Use real or containerised dependencies.

### E2E / UI tests

Drive the full stack through a browser or API client. Playwright, Cypress, Selenium. Run last; slowest.

---

## Slide 08 — Quality Gates

A **quality gate** is a threshold that the pipeline enforces. If the code does not meet the standard, the pipeline fails and the change cannot proceed.

### Coverage gate

Require minimum test coverage — e.g. 80% line or branch coverage. Tools: `pytest-cov`, `nyc`, `jacoco`.

```
coverage: 82.4%  Pass (>= 80%)
branch:   76.1%  FAIL (< 75%)  -> FAIL
```

### Static analysis gate

Run SAST tools: `semgrep`, `bandit`, `eslint`, `SonarQube`. Block merges that introduce new high-severity findings.

### Dependency vulnerability gate

Scan for known CVEs in dependencies. `npm audit`, `pip-audit`, `Snyk`, `Dependabot`. Fail on HIGH or CRITICAL findings.

### Tip

Start with loose gates and tighten over time. Enforcing 80% coverage on a legacy codebase from day one is demoralising — begin at your current level and ratchet upward.

---

## Slide 09 — Branching Strategies & Triggers

How you branch determines when and what the pipeline runs.

### Trunk-based development

Developers commit directly to `main` (or via very short-lived branches). CI runs on every push. Favoured by Google, Meta. Requires feature flags for incomplete work.

### GitHub Flow

`main` is always deployable. Work happens in feature branches, merged via pull request. CI runs on each PR and on merge to `main`. Simple and effective for most teams.

### GitFlow

Separate `develop`, `release`, and `hotfix` branches. More overhead — now considered heavyweight for most modern software.

### Common pipeline triggers

| Trigger | Typical pipeline |
|---------|-----------------|
| Push to feature branch | Build + unit tests |
| Pull request opened | Full CI (build + all tests + analysis) |
| Merge to `main` | Full CI + deploy to staging |
| Tag push (`v1.2.0`) | Full CI + deploy to production |
| Scheduled (nightly) | Slow tests, security scans |

Don't run the full slow suite on every commit to a feature branch — developers lose patience and disable CI. Run a fast subset; run everything on merge.

---

## Slide 10 — Artefacts & Environments

A **build artefact** is the immutable, versioned output of the build stage. It is promoted through environments — never rebuilt.

### Artefact types

- Docker image — tagged with git SHA or semver
- Python wheel / npm tarball
- Compiled binary or JAR
- Terraform plan

### Build once, deploy many.

Never rebuild the artefact for staging vs production. Rebuilding introduces the risk that the artefact that passed testing is not what gets deployed.

### Promotion flow

```
Build -> artefact v1.4.2-abc1234
        |
        v
Deploy to DEV — automated smoke test
        |
        v
Deploy to STAGING — E2E test suite
        |
        v
Manual approval (Continuous Delivery)
        |
        v
Deploy to PRODUCTION
```

---

## Slide 11 — Docker in CI/CD

Containers solve the **"works on my machine"** problem. A Docker image bundles the application and its entire runtime — the same image runs in CI, staging, and production.

### Multi-stage Dockerfile (Python example)

```dockerfile
# Multi-stage Dockerfile (Python example)
FROM python:3.12-slim AS builder
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

FROM python:3.12-slim AS runtime
WORKDIR /app
COPY --from=builder /usr/local/lib/python3.12 \
     /usr/local/lib/python3.12
COPY src/ .
RUN useradd -m appuser && chown -R appuser /app
USER appuser
CMD ["python", "main.py"]
```

Multi-stage builds keep the final image lean — the builder stage (with compilers and dev tools) is discarded. Runtime images should contain only what is needed to run.

### CI pipeline with Docker

- Build image: `docker build -t app:$SHA .`
- Run tests inside the container
- Scan image: `trivy image app:$SHA`
- Push to registry: `ghcr.io`, ECR, Docker Hub
- Deploy: pull image by immutable SHA tag

### Image tagging strategy

- `:latest` — avoid in production; ambiguous
- `:abc1234` — git SHA, fully traceable
- `:1.4.2` — semver for releases
- `:main-20260306` — branch + date

---

## Slide 12 — Tool Deep Dive: GitHub Actions

### Example workflow

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - uses: actions/setup-python@v5
        with:
          python-version: "3.12"

      - name: Install dependencies
        run: pip install -r requirements.txt

      - name: Lint
        run: ruff check .

      - name: Unit tests
        run: pytest tests/unit --cov=src \
             --cov-fail-under=80

      - name: Build Docker image
        run: |
          docker build \
            -t ghcr.io/${{ github.repository }}:${{ github.sha }} .

      - name: Push to registry
        if: github.ref == 'refs/heads/main'
        run: docker push \
          ghcr.io/${{ github.repository }}:${{ github.sha }}
```

### Key concepts

- **Workflow** — a YAML file in `.github/workflows/`
- **Job** — a set of steps on a single runner
- **Step** — a shell command or a reusable Action
- **Runner** — the VM executing the job
- **Action** — packaged, reusable step from the Marketplace

### Parallel jobs

Split slow test suites across multiple jobs that run concurrently. Use `needs:` to express dependencies between jobs.

---

## Slide 13 — CI/CD Tool Landscape

| Tool | Model | Config file | Strengths | Considerations |
|------|-------|-------------|-----------|----------------|
| **GitHub Actions** | SaaS / self-hosted | `.github/workflows/*.yml` | Tight GitHub integration, huge Marketplace, free for public repos | Costs scale with minutes on private repos |
| **GitLab CI/CD** | SaaS / self-hosted | `.gitlab-ci.yml` | All-in-one DevOps platform, strong environments & review apps | GitLab hosting required (or self-host) |
| **Jenkins** | Self-hosted | `Jenkinsfile` (Groovy) | Fully configurable, huge plugin ecosystem, runs anywhere | High operational burden; Groovy DSL has a learning curve |
| **CircleCI** | SaaS / self-hosted | `.circleci/config.yml` | Fast, good caching, orbs for reusable config | Costs can surprise at scale |
| **Tekton / ArgoCD** | Kubernetes-native | CRD YAML manifests | Cloud-native, GitOps-friendly, very scalable | Steep learning curve; requires Kubernetes |

For a new project on GitHub, start with **GitHub Actions**. For an enterprise self-hosted requirement with Kubernetes, evaluate **Tekton + ArgoCD**.

---

## Slide 14 — Secrets & Pipeline Security

Pipelines need credentials — API keys, registry passwords, cloud credentials. Mishandling them is one of the most common CI/CD security failures.

### Never do this

- Hardcode secrets in the YAML config
- Print secrets with `echo $MY_SECRET`
- Commit `.env` files with real values
- Use personal tokens in shared pipelines

### Correct approach

- Store secrets in the platform vault (GitHub Secrets, GitLab Variables, AWS SSM)
- Reference via `${{ secrets.MY_KEY }}` — redacted from logs
- Use OIDC / workload identity — no long-lived secrets at all
- Rotate secrets regularly; restrict to only the jobs that need them

### Supply chain security

- Pin third-party Actions to a full commit SHA, not a tag
- Use `SLSA` provenance and signed artefacts
- Scan dependencies on every build

### Least privilege runners

Runners should have the minimum IAM permissions needed. Use ephemeral runners (a fresh VM per job) rather than long-lived shared agents.

### OIDC

GitHub Actions supports OIDC federation with AWS, GCP, and Azure. The runner gets a short-lived token automatically — no stored secret needed.

---

## Slide 15 — Deployment Strategies

How you deploy to production determines your blast radius and rollback speed.

### Big-bang / recreate

Stop old version, start new version. Simple but causes downtime. Only for non-critical systems.

### Rolling update

Replace instances one at a time. Zero downtime. Old and new versions briefly coexist — APIs must be backwards-compatible.

### Blue/Green

Two identical environments. Route traffic from blue (old) to green (new). Instant rollback by switching the load balancer back. Doubles infrastructure cost briefly.

### Canary release

Route a small percentage (e.g. 5%) of traffic to the new version. Monitor error rates and latency. Gradually shift 100% if healthy, or roll back quickly if not.

### Feature flags

Deploy code but hide it behind a runtime flag. Decouple deployment from feature release. Allows trunk-based development with incomplete features safely in production.

Canary + feature flags is the combination used by most high-velocity teams (Netflix, Spotify, GitHub itself).

---

## Slide 16 — Observability & Rollback

Deploying fast is only safe if you can **detect failures quickly** and **recover faster than you broke things**.

### The three pillars

- **Metrics** — latency, error rate, saturation (Prometheus / Grafana / Datadog)
- **Logs** — structured JSON logs shipped to Loki / ELK / Splunk
- **Traces** — distributed tracing across services (OpenTelemetry, Jaeger, Tempo)

### Automated rollback

Configure alerts on key SLIs. If error rate exceeds threshold within a defined window after deploy, trigger automatic rollback — re-deploy the previous artefact SHA.

### Post-deploy checklist

| Time | Action |
|------|--------|
| **T+0m** | Deploy completes. Automated smoke tests run. |
| **T+5m** | Monitor p50/p99 latency and error rate. Compare to pre-deploy baseline. |
| **T+15m** | Check application logs for new error patterns. |
| **T+30m** | Mark deploy as stable. Close the change window. |

---

## Slide 17 — Keeping Pipelines Fast

A slow pipeline is a pipeline that developers **work around**. Target **< 10 minutes** for the core CI loop.

### Caching

Cache dependency downloads between runs. Most platforms support keying the cache on the lockfile hash.

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.npm
    key: npm-${{ hashFiles('**/package-lock.json') }}
```

### Parallelism

Split test suites into shards. Run lint, unit tests, and security scans as parallel jobs. Use a matrix strategy for multi-version or multi-OS testing.

### Path filtering

Only trigger expensive jobs when relevant files change. Skip the full test suite if only documentation changed.

```yaml
on:
  push:
    paths:
      - 'src/**'
      - 'tests/**'
      - 'requirements*.txt'
```

### Layer caching for Docker

Order Dockerfile instructions from *least* to *most* frequently changing. Copy and install dependencies before copying application code.

---

## Slide 18 — Measuring CI/CD Effectiveness: DORA Metrics

The **DORA** (DevOps Research and Assessment) team identified four metrics that predict software delivery performance.

### Deployment Frequency

How often does the team successfully deploy to production? Elite teams deploy multiple times per day. Frequent, small deployments are safer than infrequent large ones.

### Lead Time for Changes

Time from code committed to code running in production. Elite: < 1 hour. Measures the efficiency of the entire pipeline, including review and approvals.

### Change Failure Rate

Percentage of deployments that cause a production failure requiring rollback or hotfix. Elite: 0-5%. A high rate indicates inadequate testing or deployment strategy.

### Failed Deployment Recovery Time

How long to restore service after a failure. Elite: < 1 hour. Requires fast detection (observability), fast rollback, and on-call processes.

These metrics are positively correlated with business outcomes — teams in the elite tier have 127x faster lead time than low performers (DORA State of DevOps Report).

---

## Slide 19 — Summary & Next Steps

### What we covered

- Why CI/CD exists — eliminating integration risk and manual toil
- CI: automated build and test on every commit
- CD: always-releasable code (Delivery) or fully automatic release (Deployment)
- Pipeline stages, artefact promotion, environment strategy
- Testing pyramid and quality gates
- Docker, GitHub Actions, secrets security
- Deployment strategies and observability
- DORA metrics for measuring improvement

### Recommended next steps

1. Add a `.github/workflows/ci.yml` to your next project
2. Set up unit tests with a coverage gate
3. Containerise your app with a multi-stage Dockerfile
4. Add a staging environment and promote artefacts, not code
5. Instrument with Prometheus + Grafana and set a rollback alert
6. Measure your DORA metrics — establish a baseline

### Further reading

- Humble & Farley, *Continuous Delivery* (2010)
- Kim et al., *The DevOps Handbook* (2nd ed. 2021)
- DORA State of DevOps Reports
- Martin Fowler's *ContinuousIntegration* article (martinfowler.com)
