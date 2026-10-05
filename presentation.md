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
9. [Quality Gates for Code and Web Pages](#slide-09--quality-gates-for-code-and-web-pages)
10. [Tests That Guard the Results](#slide-10--tests-that-guard-the-results)
11. [Branching Strategies & Triggers](#slide-11--branching-strategies--triggers)
12. [Protecting main: Rules, Required Checks and Merges](#slide-12--protecting-main-rules-required-checks-and-merges)
13. [Artefacts & Environments](#slide-13--artefacts--environments)
14. [Docker in CI/CD](#slide-14--docker-in-cicd)
15. [Tool Deep Dive: GitHub Actions](#slide-15--tool-deep-dive-github-actions)
16. [CI/CD Tool Landscape](#slide-16--cicd-tool-landscape)
17. [CI Across Repositories](#slide-17--ci-across-repositories)
18. [Secrets & Pipeline Security](#slide-18--secrets--pipeline-security)
19. [Deployment Strategies](#slide-19--deployment-strategies)
20. [Deploying Safely: Previews, Settings and Migrations](#slide-20--deploying-safely-previews-settings-and-migrations)
21. [After the Deploy: Smoke Checks and Health Endpoints](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints)
22. [Observability & Rollback](#slide-22--observability--rollback)
23. [Case Study: Two Failures CI Did Not Catch](#slide-23--case-study-two-failures-ci-did-not-catch)
24. [Keeping Pipelines Fast](#slide-24--keeping-pipelines-fast)
25. [Measuring CI/CD Effectiveness: DORA Metrics](#slide-25--measuring-cicd-effectiveness-dora-metrics)
26. [Key Terms (1 of 2)](#slide-26--key-terms-1-of-2)
27. [Key Terms (2 of 2)](#slide-27--key-terms-2-of-2)
28. [Summary & Next Steps](#slide-28--summary--next-steps)

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
- Protecting `main`; CI across repositories

### Testing in CI

- Test pyramid
- Unit, integration, E2E tests
- Code coverage and quality gates
- Lint, types, Lighthouse; tests that pin results

### Tools & Practice

- GitHub Actions, GitLab CI, Jenkins
- Docker in CI/CD
- Secrets and security
- Deploying safely: migrations, smoke checks, health endpoints
- Observability and rollbacks; a case study; key terms

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

More gates: lint, types, formatting and Lighthouse on [slide 09](#slide-09--quality-gates-for-code-and-web-pages); tests that pin results on [slide 10](#slide-10--tests-that-guard-the-results).

---

## Slide 09 — Quality Gates for Code and Web Pages

Tests prove behaviour. These cheaper gates catch the rest in seconds, before a reviewer reads a line.

### Static checks

- **Lint**: a linter (ESLint, ruff, clippy) flags suspicious or banned code patterns without running the code.
- **Typecheck**: the compiler checks every type without building (`tsc --noEmit`), so a wrong argument fails CI, not production.
- **Format check**: a formatter (Prettier, `cargo fmt`, `ruff format`) in check mode fails if any file is not formatted, so reviews never argue about layout.
- **actionlint** lints the workflow files themselves ([GitHub Actions deck, slide 16](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/16)).

### Lighthouse and budgets

- **Lighthouse**: Google's page auditor. It loads a page in headless Chrome and scores performance, accessibility, best practices and SEO from 0 to 100.
- **Lighthouse CI (LHCI)**: runs Lighthouse inside the pipeline on chosen URLs, several times, and fails the job when an assertion is not met.
- **Performance budget**: the limit a gate enforces, either a minimum score (≥ 0.9) or a maximum cost (script size, LCP time).
- **Core Web Vitals**: Google's three measures of real visits. LCP (loading, good ≤ 2.5 s), INP (response to input, ≤ 200 ms) and CLS (layout shift, ≤ 0.1), judged at the 75th percentile.

### A real gate: three pages, three runs each

```json
"assertions": {
  "categories:performance": ["error", { "minScore": 0.9 }],
  "categories:accessibility": ["error", { "minScore": 0.9 }],
  "categories:best-practices": ["error", { "minScore": 0.9 }]
}
```

From a Next.js site on this GitHub: its `Lighthouse` job fails the PR if the median score of any category drops below 0.9.

### Hedges

- A lab run has no real user input, so it cannot measure INP. Lighthouse uses Total Blocking Time instead.
- Scores move from run to run, and the weights change between versions (v10+: TBT 30 %, LCP 25 %, CLS 25 %, FCP 10 %, Speed Index 10 %). Gate on a median, and check the [scoring docs](https://developer.chrome.com/docs/lighthouse/performance/performance-scoring) and [web.dev/vitals](https://web.dev/articles/vitals).

---

## Slide 10 — Tests That Guard the Results

Simulators and numeric code need tests that pin numbers, not only behaviour. Every example here runs in CI on this GitHub.

### Pinning the numbers

- **Test fixture**: the fixed inputs a test runs against (a saved trace, a config, a seeded database). Commit them: one hidden by `.gitignore` failed CI on 3 Oct 2026. An **up-to-date check** is CI regenerating such files and failing on any difference.
- **Golden test** (or **headline-number check**): output must equal a saved, reviewed answer. FHE_Accelerator_Sim's CI asserts its ARK-class baseline is still 13.94 ms per bootstrap; changing it is a deliberate re-bless.
- **Parity test**: two implementations of one model must agree. The JavaScript port of Disaggregated_Inference_Sim is **bit-exact** with the Python (identical floats); only transcendental functions get a tolerance.
- **Differential test**: the same random inputs go through two implementations and the outputs are compared (Rust_DES_Kernel against the Python simulator).

### Gating on speed

- **Performance regression gate**: the build fails if a benchmark slows beyond a threshold, set wider than the run-to-run noise you measured ([SimEng 07, slide 08](https://brendanjameslynskey.github.io/SimEng_07_Jenkins_for_Simulation_Teams/#slide-08)).

### Browser tests that mean it

- **Playwright**: a test runner that drives real Chromium, Firefox and WebKit through the app, as a user would.
- **e2e on a production build**: run the end-to-end tests against `next build` + `next start`, not the dev server, so the bundle that ships is the one tested.
- **Flaky test**: a test that passes and fails on the same code. Find the race (usually a missing wait) rather than adding retries. One e2e test read a metrics page before its comment POST had finished, and failed once in CI.

### Searching for bugs

- **Property-based test**: you state a rule ("tokens out equal tokens in") and a tool (Hypothesis, proptest) generates hundreds of inputs, shrinking any failure to the smallest case.
- **Mutation testing**: a tool (mutmut, cargo-mutants) plants small bugs; any that the tests miss mark a blind spot. A stricter measure than coverage.

Go deeper: [Testing Frameworks for Simulators](https://brendanjameslynskey.github.io/SimEng_06_Testing_Frameworks/) (fixtures, Hypothesis, golden tests, mutation testing).

---

## Slide 11 — Branching Strategies & Triggers

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

## Slide 12 — Protecting main: Rules, Required Checks and Merges

`main` is what ships. Protection rules turn the pipeline's verdict from advice into a rule.

### The rules

- **Pull request (PR)**: a request to merge a branch into `main`. CI runs on it and reviewers comment before anything lands (GitLab calls it a merge request).
- **Branch protection**: per-branch settings that stop direct pushes and set the conditions for merging.
- **Ruleset**: GitHub's newer form of the same rules. Several can apply to one branch (the strictest wins), they can be disabled without being deleted, and anyone who can read the repo can see them.
- **Required status check**: a named CI check (on GitHub, the job's name) that must pass before the PR can merge. Rename the job and the rule waits for the old name forever.
- **Required reviews** and **CODEOWNERS**: a number of approvals before merging; a `CODEOWNERS` file names who must review which paths.

### What they stop, and who may skip them

- **Force push**: `git push --force` replaces the remote branch's history. Protection blocks it on `main`, and blocks deleting the branch.
- **Bypass list** (rulesets) or **admin enforcement** (branch protection): whether the rules bind administrators too. With an empty bypass list even the owner's push is refused: `GH013 … Changes must be made through a pull request`.

### Merging and showing the state

- **Squash merge**: the PR's commits become one commit on `main`, one per change.
- **Green at HEAD**: the newest commit on the branch (its HEAD) has passed every check. "It was green earlier" does not count.
- **CI status badge**: an image in the README showing the latest run's result on `main`.

On this GitHub: a Next.js site requires five checks (Lint & Typecheck, Unit Tests, Verify maths against PyTorch fixtures, E2E Tests, Lighthouse); the GitHub Actions deck runs a ruleset with an empty bypass list and shows a PR blocked ([slides 19](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/19), [20](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/20)). Plans differ for private repos: [about rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/about-rulesets), [code owners](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/customizing-your-repository/about-code-owners).

---

## Slide 13 — Artefacts & Environments

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

## Slide 14 — Docker in CI/CD

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

## Slide 15 — Tool Deep Dive: GitHub Actions

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

### Go deeper

[Introduction to GitHub Actions](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/): matrices, caching, reuse, security, debugging and protecting `main`, every example run for real.

---

## Slide 16 — CI/CD Tool Landscape

| Tool | Model | Config file | Strengths | Considerations |
|------|-------|-------------|-----------|----------------|
| **GitHub Actions** | SaaS / self-hosted | `.github/workflows/*.yml` | Tight GitHub integration, huge Marketplace, free for public repos | Costs scale with minutes on private repos |
| **GitLab CI/CD** | SaaS / self-hosted | `.gitlab-ci.yml` | All-in-one DevOps platform, strong environments & review apps | GitLab hosting required (or self-host) |
| **[Jenkins](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/)** | Self-hosted | `Jenkinsfile` (Groovy) | Fully configurable, huge plugin ecosystem, runs anywhere | High operational burden; Groovy DSL has a learning curve |
| **CircleCI** | SaaS / self-hosted | `.circleci/config.yml` | Fast, good caching, orbs for reusable config | Costs can surprise at scale |
| **Tekton / ArgoCD** | Kubernetes-native | CRD YAML manifests | Cloud-native, GitOps-friendly, very scalable | Steep learning curve; requires Kubernetes |

For a new project on GitHub, start with **GitHub Actions**. For an enterprise self-hosted requirement with Kubernetes, evaluate **Tekton + ArgoCD**.

---

## Slide 17 — CI Across Repositories

When one repo installs another, a push to the library can break the app without a line of the app changing.

### Upstream and downstream

- **Downstream (dependent) repo**: a repo that installs yours. Its CI is the real test of an upstream change. Here, Rust_DES_Kernel and Torch_Sim_Frontend install Disaggregated_Inference_Sim; RTL_CoSim_NTT, SystemC_Accelerator_Model and Memory_System_Sim install FHE_Accelerator_Sim.
- **Tracking a branch vs pinning**: `pkg @ git+URL` installs whatever `main` is today; `git+URL@<sha>` pins one commit. Tracking catches breakage at once but can turn a green repo red overnight. Pinning is reproducible but needs deliberate bumps.
- **Re-running a workflow**: runs the same commit's checks again (same `GITHUB_SHA`, within 30 days). An unpinned install resolves again, so a re-run tests the new upstream: `gh run rerun <id>`, or `gh workflow run ci.yml` where the workflow has a manual trigger.

### Copying instead of installing

- **Vendoring**: copying another repo's file into yours from a pinned commit, with the commit and SHA-256 recorded (`VENDORED.json`) so a test can check the copy.
- **Scheduled drift check**: a weekly cron job that re-fetches pinned upstream files and flags any that changed, so re-pinning is a decision.

### Keeping installs honest

- **Lockfile**: a file recording the exact version of every dependency (`pnpm-lock.yaml`, `Cargo.lock`). `pnpm install --frozen-lockfile` fails rather than quietly changing it in CI.
- **Stale environment**: pip skips a `pkg @ git+…` that is already installed, so a reused venv on a long-lived agent keeps testing the old upstream. The Jenkinsfiles force it:

```
.venv/bin/pip install -q --force-reinstall --no-deps "disagg-sim @ git+https://github.com/BrendanJamesLynskey/Disaggregated_Inference_Sim"
```

### Automating it

- The upstream's CI can start the downstream's with `repository_dispatch` ([GitHub Actions deck, slide 23](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/23)); Dependabot can open the pin bumps ([slide 14](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/14)).

Re-run rules: [docs.github.com](https://docs.github.com/en/actions/how-tos/manage-workflow-runs/re-run-workflows-and-jobs). pip VCS installs: [pip docs](https://pip.pypa.io/en/stable/topics/vcs-support/).

---

## Slide 18 — Secrets & Pipeline Security

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

## Slide 19 — Deployment Strategies

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

## Slide 20 — Deploying Safely: Previews, Settings and Migrations

### Where code goes

- **Preview deployment**: a throwaway copy of the app built from a branch or PR, with its own URL, for checking before merge. **Production deployment**: the one the real domain serves. On Vercel, by default, the production branch (usually `main`) deploys to production and other branches and PRs get previews.
- **Git-integrated vs CLI deploy**: with Git integration the platform deploys on every push. A CLI deploy (`vercel deploy --prod`) ships the folder you run it in, so deploy a **clean export** (`git archive HEAD`), not a working tree with local files in it.
- **GitHub Pages**: GitHub serves a branch (or a workflow's output) as a static site. These decks publish that way: push to `main` and the site updates within minutes.
- **Deployment protection**: a sign-in gate on preview URLs (Vercel Authentication), so a smoke check gets a 401 or a login page. A **protection bypass** token in the `x-vercel-protection-bypass` header lets automation through; keep it secret.
- **Custom domain**: your own name for the production site. **Redirect (308)**: when the domain moves, the old URL should answer 308 Permanent Redirect, which keeps the method and body (a 301 may turn a POST into a GET).

### Settings that live outside git

- **Environment variable**: a setting the app reads at run time (`DATABASE_URL`), kept out of git and set separately for development, preview and production. Passwords and keys among them are secrets.
- **Sensitive variable**: a write-only variable that cannot be read back after saving, so `vercel env pull` returns it empty (Vercel now calls this type "Secret").
- **OAuth callback URL**: where the identity provider sends the user after sign-in. It must match the deployed domain, so a domain move means updating the OAuth app.

### Changing the database

- **Schema migration**: a versioned script that changes the database's structure (a table, a column), applied in order. **Seeding**: loading starter rows after migrating.
- **Migrate before deploy**: run the new migration on production first, then deploy the code that needs it. The old code keeps running in between.
- **Expand/contract**: what makes that safe. Expand (add nullable columns or new tables that old code ignores), deploy, backfill, then contract (drop the old) in a later release.

[Vercel environments](https://vercel.com/docs/deployments/environments) · [deployment protection](https://vercel.com/docs/deployment-protection) · [protection bypass](https://vercel.com/docs/deployment-protection/methods-to-bypass-deployment-protection/protection-bypass-automation) · [write-only variables](https://vercel.com/docs/environment-variables/sensitive-environment-variables) · [HTTP 308](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/308) · [expand/contract (Fowler)](https://martinfowler.com/bliki/ParallelChange.html)

---

## Slide 21 — After the Deploy: Smoke Checks and Health Endpoints

CI tested a copy. Production has its own database, runtime and settings, so check the live system too.

### Checking the live system

- **Post-deploy verification**: the checks run against the live URL straight after a deploy, before anyone calls it done.
- **Health endpoint**: a URL that reports whether the app's dependencies work. It answers 200 when healthy and 503 when, say, the database is down or its schema is behind the code. Load balancers and Kubernetes probes poll the same kind of URL.
- **Smoke check**: a fast, shallow test of the deployed system: every page answers 200 and the health endpoint is healthy. (The name comes from hardware: switch on, and check nothing smokes.) It exits non-zero on any failure.
- **Smoke test on real data**: an empty list never runs the rendering code, so the check fetches a list that really holds an item and requires it to render.

### A live health reply (4 Oct 2026)

```
GET /api/health → 200
{"ok":true,"data":{"healthy":true,"db":"up","dbError":null,
 "schema":{"status":"current","applied":1,"expected":1, …}}}
```

### When something is wrong

- **Runtime logs**: the platform's record of live requests and errors. **Error grouping** shows one bug hit 14 times as one entry, not 14 lines. On Vercel's Hobby plan the CLI reached back only about an hour here; the dashboard keeps grouped errors for days.
- **Runbook**: the written, step-by-step procedure for an operation (deploy, restore, rotate a key), so it is done the same way every time, by anyone. This site's deploy runbook: migrate, deploy, smoke-check, read the logs.
- **Render before write**: do every step that can fail before storing anything. Otherwise a crash after the write leaves a stored comment and an error; the user retries, and the comment appears three times.

Health-check patterns: [Kubernetes probes](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/). The case study on slide 23 shows why each of these exists.

---

## Slide 22 — Observability & Rollback

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

## Slide 23 — Case Study: Two Failures CI Did Not Catch

Both happened in 2026 to a small Next.js + Postgres site on Vercel, built from this GitHub. Both had green CI.

### 1. The database nobody migrated

- To let a fresh clone run without a database, the data layer had a **silent fallback**: an error swallowed and replaced by a default (here, empty data), so the failure never shows.
- Production's database had never been migrated. Every query failed, every page showed empty lists, and nothing reported an error, for months.
- **Now caught by**: a health endpoint that refuses to fall back (503 for a schema that is behind or missing), a migrate-first step in the runbook, and the post-deploy smoke check. The admin page also counts fallbacks, so a recent failure stays visible.

Lesson: every fallback needs a signal that it happened.

### 2. Works locally, fails on the platform

- **Works locally, fails on the platform**: production's runtime differs from the one CI used (here, the module loader).
- **Module loader**: the part of a runtime that finds and loads packages. Node can `require()` an ES module (a package written with `import`/`export`); some platform loaders cannot.
- A comment sanitiser pulled in a package that is an ES module only, loaded with `require()`. Node 20.19+ and 22.12+ allow that, so local runs and CI passed. The platform's function loader refused (`ERR_REQUIRE_ESM`), and every comment list with a comment in it returned 500: 14 errors in under 3 minutes.
- **Now caught by**: CI's end-to-end server runs under a flag that makes Node refuse the same thing, so the bug would fail the PR:

```
command:
  "pnpm build && node --no-experimental-require-module node_modules/next/dist/bin/next start -p 3000",
```

- Also: a sanitiser the bundler can package, render before write, and a smoke check that reads a real comment.

[Node: require() of ES modules](https://nodejs.org/api/modules.html#loading-ecmascript-modules-using-require)

---

## Slide 24 — Keeping Pipelines Fast

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

## Slide 25 — Measuring CI/CD Effectiveness: DORA Metrics

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

## Slide 26 — Key Terms (1 of 2)

Every CI/CD term used on this GitHub's projects, with the slide that explains it. Bare numbers are slides of this deck; GHA = [Introduction to GitHub Actions](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/), Jenkins = [Introduction to Jenkins](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/).

| Term | Plain meaning | Explained on |
|------|---------------|--------------|
| Pull request (PR) | a proposed merge, where CI and review happen | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Branch protection | rules that guard a branch from direct pushes | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Ruleset | GitHub's newer, layered, visible branch rules | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Required status check | a named check that must pass to merge | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Required reviews | approvals needed before merging | [GHA 19](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/19) |
| CODEOWNERS | file naming who must review which paths | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Force push | rewriting a branch's history on the remote | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Bypass list / admin enforcement | who, if anyone, may skip the rules | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Squash merge | a PR lands as one commit | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Green at HEAD | the latest commit passed every check | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| CI status badge | README image of the latest run's result | [12](#slide-12--protecting-main-rules-required-checks-and-merges) |
| Dependabot | GitHub's bot that opens dependency-bump PRs | [GHA 14](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/14) |
| workflow_dispatch | a manual Run workflow trigger | [GHA 04](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/4) |
| Re-running a workflow | running the same commit's checks again | [17](#slide-17--ci-across-repositories) |
| Downstream (dependent) repo | a repo that installs yours | [17](#slide-17--ci-across-repositories) |
| Pinning to a commit | referencing one immutable SHA, not a branch | [GHA 14](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/14) |
| repository_dispatch | one repo's CI starting another's | [GHA 23](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/23) |
| Webhook | the forge calls CI when something happens | [Jenkins 10](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/#/10) |
| Scheduled (nightly) build | a run on a timetable, not a push | [Jenkins 10](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/#/10) |
| Concurrency group | runs that must not overlap | [GHA 18](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/18) |
| Lockfile / frozen install | exact versions; CI refuses to change them | [17](#slide-17--ci-across-repositories) |
| Stale environment | a reused venv keeps an old dependency | [17](#slide-17--ci-across-repositories) |
| Vendoring | a pinned, checked copy of another repo's file | [17](#slide-17--ci-across-repositories) |
| Scheduled drift check | a timed run that flags upstream changes to pins | [17](#slide-17--ci-across-repositories) |
| Pipeline as code | the pipeline lives in the repo as a file | [05](#slide-05--the-cicd-pipeline) |
| Runner / agent | the machine a job runs on | [15](#slide-15--tool-deep-dive-github-actions) |
| Matrix build | one job run per combination of values | [GHA 06](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/6) |
| Cache / cache key | saved downloads reused by later runs | [24](#slide-24--keeping-pipelines-fast) |
| Artefact | a build output stored and promoted | [13](#slide-13--artefacts--environments) |
| Service container | a database container beside the job | [GHA 07](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/7) |
| Deployment environment | a named target with its own secrets/rules | [GHA 13](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/13) |
| Shared library | pipeline code shared by many repos | [Jenkins 14](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/#/14) |
| Multibranch pipeline | one Jenkins job per branch and PR | [Jenkins 15](https://brendanjameslynskey.github.io/Introduction_to_Jenkins/#/15) |
| Publishing to GitHub Pages | a branch served as a static site | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Quality gate | a threshold the pipeline enforces | [08](#slide-08--quality-gates) |
| Lint | flag suspicious code without running it | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Typecheck | the compiler checks every type | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Format check | fail if any file isn't formatted | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| actionlint | a linter for workflow files | [GHA 16](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/16) |
| Lighthouse | Google's page auditor, scores 0-100 | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Lighthouse CI (LHCI) | Lighthouse in CI, failing below a score | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Performance budget | the score or size limit a gate enforces | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Core Web Vitals | LCP, INP, CLS: real users' experience | [09](#slide-09--quality-gates-for-code-and-web-pages) |
| Coverage target | minimum share of code the tests run | [08](#slide-08--quality-gates) |
| Unit / integration test | one unit alone / parts together | [07](#slide-07--the-test-pyramid) |

---

## Slide 27 — Key Terms (2 of 2)

| Term | Plain meaning | Explained on |
|------|---------------|--------------|
| End-to-end (e2e) test | drive the whole app like a user | [07](#slide-07--the-test-pyramid) |
| Playwright | a browser-automation test runner | [10](#slide-10--tests-that-guard-the-results) |
| e2e on a production build | test the bundle that ships | [10](#slide-10--tests-that-guard-the-results) |
| Test fixture | fixed inputs a test runs against | [10](#slide-10--tests-that-guard-the-results) |
| Golden / headline-number check | output must equal a reviewed answer | [10](#slide-10--tests-that-guard-the-results) |
| Parity test (bit-exact) | two implementations must agree exactly | [10](#slide-10--tests-that-guard-the-results) |
| Differential test | same random inputs through two versions | [10](#slide-10--tests-that-guard-the-results) |
| Up-to-date check | CI regenerates outputs and fails on a difference | [10](#slide-10--tests-that-guard-the-results) |
| Property-based test | generated inputs against a stated rule | [10](#slide-10--tests-that-guard-the-results) |
| Mutation testing | planted bugs the tests should catch | [10](#slide-10--tests-that-guard-the-results) |
| Flaky test | passes and fails on the same code | [10](#slide-10--tests-that-guard-the-results) |
| Performance regression gate | fail if a benchmark slows past noise | [10](#slide-10--tests-that-guard-the-results) |
| Silent fallback | an error swallowed and replaced by a default | [23](#slide-23--case-study-two-failures-ci-did-not-catch) |
| Preview deployment | a throwaway copy built from a branch | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Deployment protection / protection bypass | a login gate on deployment URLs; a token lets checks through | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Production deployment | the one the real domain serves | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Git-integrated vs CLI deploy | deploy on push / deploy a folder | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Clean export (git archive) | deploy exactly the commit, nothing local | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Environment variable | a setting the app reads at run time | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Secret / masking | encrypted setting, shown as *** in logs | [GHA 13](https://brendanjameslynskey.github.io/Introduction_to_GitHub_Actions/#/13) |
| Sensitive (write-only) variable | can't be read back after saving | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| OAuth callback URL | where sign-in returns the user | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Custom domain | your own name for the production site | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Redirect (308) | old URL permanently forwards to the new | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| OIDC | short-lived cloud credentials, no stored key | [18](#slide-18--secrets--pipeline-security) |
| Least privilege | grant only the access a job needs | [18](#slide-18--secrets--pipeline-security) |
| Schema migration | a versioned change to the database | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Seeding | loading starter rows after migrating | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Migrate before deploy | schema change first, then new code | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Expand/contract | add, switch, then remove, across releases | [20](#slide-20--deploying-safely-previews-settings-and-migrations) |
| Blue/green | two environments, switch traffic | [19](#slide-19--deployment-strategies) |
| Canary release | a small share of traffic first | [19](#slide-19--deployment-strategies) |
| Feature flag | code shipped dark, switched on at run time | [19](#slide-19--deployment-strategies) |
| Rollback | return to the previous good version | [22](#slide-22--observability--rollback) |
| Post-deploy verification | checking the live system after a deploy | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Health endpoint | URL reporting if dependencies are OK | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Smoke check | fast, shallow test of the live system | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Smoke test on real data | exercise code paths empty data skips | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Runtime logs / error groups | the platform's record of live errors | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Runbook | the written procedure for an operation | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Render before write | do what can fail before storing | [21](#slide-21--after-the-deploy-smoke-checks-and-health-endpoints) |
| Works locally, fails on the platform | production's runtime differs from CI's | [23](#slide-23--case-study-two-failures-ci-did-not-catch) |
| Module loader (ESM vs require) | how a runtime loads JS packages | [23](#slide-23--case-study-two-failures-ci-did-not-catch) |
| DORA metrics | four delivery-performance measures | [25](#slide-25--measuring-cicd-effectiveness-dora-metrics) |

---

## Slide 28 — Summary & Next Steps

### What we covered

- Why CI/CD exists — eliminating integration risk and manual toil
- CI: automated build and test on every commit
- CD: always-releasable code (Delivery) or fully automatic release (Deployment)
- Pipeline stages, artefact promotion, environment strategy
- Testing pyramid and quality gates
- Docker, GitHub Actions, secrets security
- Protected branches, required checks, CI across repos
- Deployment strategies, migrations, smoke checks and observability
- DORA metrics for measuring improvement

### Recommended next steps

1. Add a `.github/workflows/ci.yml` to your next project
2. Set up unit tests with a coverage gate; make CI a required check on `main`
3. Containerise your app with a multi-stage Dockerfile
4. Add a staging environment and promote artefacts, not code
5. Instrument with Prometheus + Grafana, add a health endpoint and a post-deploy smoke check, and set a rollback alert
6. Measure your DORA metrics — establish a baseline

### Further reading

- Humble & Farley, *Continuous Delivery* (2010)
- Kim et al., *The DevOps Handbook* (2nd ed. 2021)
- DORA State of DevOps Reports
- Martin Fowler's *ContinuousIntegration* article (martinfowler.com)
