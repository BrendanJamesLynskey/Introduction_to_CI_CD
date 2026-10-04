# 🔁 Introduction to CI/CD

An interactive Reveal.js presentation covering Continuous Integration and Continuous Delivery/Deployment — from the problem it solves through quality gates, protecting `main`, CI across repositories, deploying safely (migrations, smoke checks, health endpoints), observability, DORA metrics, and a Key terms glossary of every CI/CD term used on this GitHub.

## ▶ [Open the Presentation](https://brendanjameslynskey.github.io/Introduction_to_CI_CD/)

## 📄 [Markdown Version](presentation.md)

---

## Contents

| # | Topic | Description |
|---|-------|-------------|
| 01 | Title | CI/CD pipeline overview |
| 02 | Agenda | Topics at a glance |
| 03 | The Problem | Integration hell, manual releases, root causes |
| 04 | Continuous Integration | Core rules, feedback loop |
| 05 | Delivery vs Deployment | Manual gate vs fully automatic — when to use each |
| 06 | The Pipeline | Stages, pipeline-as-code, key properties |
| 07 | Build Stage | Dependencies, linting, artefact creation |
| 08 | Test Pyramid | Unit, integration, and E2E tests |
| 09 | Quality Gates | Coverage, static analysis, dependency scanning |
| 10 | Quality Gates for Code and Web Pages | Lint, typecheck, format check, Lighthouse and LHCI, budgets, Core Web Vitals |
| 11 | Tests That Guard the Results | Fixtures, golden and parity tests, property-based and mutation testing, Playwright, flaky tests |
| 12 | Branching Strategies | Trunk-based, GitHub Flow, GitFlow, pipeline triggers |
| 13 | Protecting main | Branch protection, rulesets, required checks, CODEOWNERS, bypass lists, squash merges, green at HEAD |
| 14 | Artefacts & Environments | Build-once-deploy-many, environment promotion flow |
| 15 | Docker in CI/CD | Multi-stage builds, image tagging strategy |
| 16 | GitHub Actions | Annotated workflow example, jobs and steps |
| 17 | Tool Landscape | GitHub Actions, GitLab CI, Jenkins, CircleCI, Tekton |
| 18 | CI Across Repositories | Downstream repos, tracking vs pinning, re-runs, lockfiles, stale environments |
| 19 | Secrets & Security | Vault patterns, OIDC, supply chain hardening |
| 20 | Deployment Strategies | Rolling, blue/green, canary, feature flags |
| 21 | Deploying Safely | Preview vs production, Git vs CLI deploys, env vars, OAuth callbacks, 308 redirects, migrate-first, expand/contract |
| 22 | After the Deploy | Post-deploy verification, health endpoints, smoke checks, runtime logs, runbooks, render before write |
| 23 | Observability & Rollback | Metrics/logs/traces, post-deploy checklist |
| 24 | Case Study | A database nobody migrated (silent fallback); green CI, red production (module loader) |
| 25 | Pipeline Optimisation | Caching, parallelism, path filtering, Docker layers |
| 26 | DORA Metrics | Deployment frequency, lead time, failure rate, recovery |
| 27 | Key Terms (1 of 2) | Every CI/CD term used on this GitHub, linked to the slide (in any CI deck) that explains it |
| 28 | Key Terms (2 of 2) | Continued |
| 29 | Summary & Next Steps | Recommended reading and practical starting points |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Big Shoulders Display + Public Sans + Overpass Mono

Single self-contained `index.html` — no npm, no dependencies to install. It and `presentation.md` are rendered from templates, so the slide numbers, Key terms links and quoted config lines (cut from pinned commits of the repos they come from) stay in step.

## See also

- [Cloud_aaS_03_PaaS_FaaS_CaaS](https://github.com/BrendanJamesLynskey/Cloud_aaS_03_PaaS_FaaS_CaaS) — what these pipelines deploy to (managed compute layers).
- [Cloud_aaS_05_Cloud_Security](https://github.com/BrendanJamesLynskey/Cloud_aaS_05_Cloud_Security) — securing the pipeline itself (OIDC federation, signed builds, SBOM, SLSA, Sigstore).
- [Introduction_to_Jenkins](https://github.com/BrendanJamesLynskey/Introduction_to_Jenkins) — a newcomer's tour of Jenkins, the self-hosted tool on the Tool Landscape slide, with real screenshots and pipelines.
- [Introduction_to_GitHub_Actions](https://github.com/BrendanJamesLynskey/Introduction_to_GitHub_Actions) — the GitHub Actions deep dive behind slide 15: workflows, caching, security, debugging and rulesets, every example run for real.
- Series hub: [Cloud `*aaS`](https://github.com/BrendanJamesLynskey/Cloud_aaS_Hub).

## References

Martin Fowler, *Continuous Integration* — martinfowler.com · Humble & Farley, *Continuous Delivery*, Addison-Wesley, 2010 · Kim, Humble, Debois & Willis, *The DevOps Handbook*, 2nd ed., 2021 · DORA, *State of DevOps Report* (annual) · Google, *DORA Metrics* — cloud.google.com/devops

## License

Educational use. Code examples provided as-is.
