# 🔁 Introduction to CI/CD

An interactive Reveal.js presentation covering Continuous Integration and Continuous Delivery/Deployment — from the problem it solves through to deployment strategies, observability, and DORA metrics.

## ▶ [Open the Presentation](./index.html)

> **Setup:** Enable GitHub Pages (Settings → Pages → Deploy from `main` branch, `/ (root)` directory), then open the Pages URL in any browser.
>
> Alternatively, open `index.html` locally in any browser — the presentation works offline after first load.

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
| 10 | Branching Strategies | Trunk-based, GitHub Flow, GitFlow, pipeline triggers |
| 11 | Artefacts & Environments | Build-once-deploy-many, environment promotion flow |
| 12 | Docker in CI/CD | Multi-stage builds, image tagging strategy |
| 13 | GitHub Actions | Annotated workflow example, jobs and steps |
| 14 | Tool Landscape | GitHub Actions, GitLab CI, Jenkins, CircleCI, Tekton |
| 15 | Secrets & Security | Vault patterns, OIDC, supply chain hardening |
| 16 | Deployment Strategies | Rolling, blue/green, canary, feature flags |
| 17 | Observability & Rollback | Metrics/logs/traces, post-deploy checklist |
| 18 | Pipeline Optimisation | Caching, parallelism, path filtering, Docker layers |
| 19 | DORA Metrics | Deployment frequency, lead time, failure rate, recovery |
| 20 | Summary & Next Steps | Recommended reading and practical starting points |

---

## Slide Controls

| Action | Key |
|--------|-----|
| Next / Previous | `→` `←` or swipe |
| Overview | `Esc` |
| Fullscreen | `F` |
| Export to PDF | Append `?print-pdf` to URL, then print |

## Technology

[Reveal.js 4.6](https://revealjs.com) · [highlight.js](https://highlightjs.org) · Playfair Display + DM Sans + JetBrains Mono

Single self-contained `index.html` — no build step, no npm, no dependencies to install.

## References

Martin Fowler, *Continuous Integration* — martinfowler.com · Humble & Farley, *Continuous Delivery*, Addison-Wesley, 2010 · Kim, Humble, Debois & Willis, *The DevOps Handbook*, 2nd ed., 2021 · DORA, *State of DevOps Report* (annual) · Google, *DORA Metrics* — cloud.google.com/devops

## License

Educational use. Code examples provided as-is.
