# CoopHub

A co-op / internship aggregator for university students. Browse listings from multiple sources in one place, search by stack / location / visa / cycle, with duplicates collapsed across sources.

> **The app is the payload — the platform around it is the project.** CoopHub is a vehicle for learning the full DevOps lifecycle end-to-end: containers, CI/CD, IaC, Kubernetes, GitOps, and observability.

---

## Architecture

```
Browser ──▶ Frontend (SPA) ──▶ FastAPI API ──▶ Postgres (truth)
                                    │
                                    └──▶ Redis (cache)
                                              ▲
                                              │
                              Worker (scrape + dedupe + upsert)
```

Four services on purpose, so orchestration / networking / scaling / monitoring all become *real* problems.

## Stack

| Layer            | Tool                                      |
| ---------------- | ----------------------------------------- |
| Language         | Python (FastAPI + worker)                 |
| Frontend         | Vite + minimal SPA                        |
| Datastores       | PostgreSQL (truth), Redis (cache)         |
| Containers       | Docker + docker-compose                   |
| CI/CD            | GitHub Actions                            |
| IaC              | Terraform                                 |
| Orchestration    | Kubernetes (kind → AWS EKS)               |
| GitOps           | ArgoCD                                    |
| Observability    | Prometheus + Grafana + Loki + OpenTelemetry |
| Security         | Trivy image scanning                      |
| Cloud            | AWS                                       |

## Build phases

- [x] **Phase 0** — Foundations & repo setup
- [ ] **Phase 1** — App + containerization (`docker compose up`)
- [ ] **Phase 2** — CI pipeline (GitHub Actions)
- [ ] **Phase 3** — Terraform (AWS VPC + EKS)
- [ ] **Phase 4** — Kubernetes (kind → EKS)
- [ ] **Phase 5** — GitOps with ArgoCD
- [ ] **Phase 6** — Observability
- [ ] **Phase 7** — Reliability & DevSecOps

## Running locally

> Will be filled in at the end of Phase 1.

## Notes for me

See [LEARNINGS.md](./LEARNINGS.md) for a running log of every concept covered, in plain English. That file is interview prep.
