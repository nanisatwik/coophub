# LEARNINGS

A running, plain-English log of every concept we cover. Read this before interviews.

---

## Phase 0 — Foundations & repo setup

### Git repository
A folder where every change is snapshotted with a "why" message. Two superpowers: time-travel to any past state, and a single source of truth that humans + machines (CI, ArgoCD) all agree on. Every DevOps tool we'll use assumes git exists.

### Branches & trunk-based workflow
`main` is always deployable. Work happens on short-lived `feat/*` branches, lands via Pull Request. The PR is the airlock where CI runs *before* code touches `main`. This matters more later: ArgoCD will deploy `main` automatically, so a broken `main` = broken prod.

### Commit hygiene
- Small commits, one logical change each.
- Subject line: imperative, <60 chars (`Add API healthcheck`, not `added stuff`).
- Subject = headline; blank line; body = the "why." Future-you at 2am will thank present-you.

### Monorepo
All services in one repo. Simpler than splitting per-service for a solo learner, and it's what Google/Meta/Uber actually do. Big upside: one PR can touch API + worker + infra atomically.

### .gitignore
Tells git what to *never* track: secrets (`.env`), generated files (`__pycache__`, `node_modules`, `*.tfstate`), and OS noise (`.DS_Store`). A leaked `.env` in git history is a recurring nightmare; the `.gitignore` is the first line of defense.

### Empty directories
Git tracks *files*, not directories. Empty folders vanish on clone. Either commit a `.gitkeep` placeholder or just don't create the folder until it has real content. We chose the latter — service folders appear in Phase 1.
