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

### Local repo vs GitHub repo
Two separate things. The `.git/` folder on your laptop = local history. GitHub = a hosted copy. `git push` syncs local → GitHub; `git pull` syncs the other way. They are only connected once you `git remote add origin <url>`. CI (Phase 2), ArgoCD (Phase 5), and PRs all require the GitHub copy.

### `origin` and `-u`
`origin` is the conventional name for the default remote (just a label, nothing magic). `git push -u origin main` pushes `main` *and* sets up tracking — after that, plain `git push` / `git pull` knows where to go.

### First-push collision + `--force-with-lease`
If you tick "Add a README" when creating a repo on github.com, the remote has a commit your local doesn't, and `git push` is rejected with "fetch first." Two ways out: `git pull --rebase` (replay your work on top, resolve conflicts) for the normal case, or `git push --force-with-lease` to overwrite remote — *only* safe when the remote has nothing worth keeping and you're the only contributor. `--force-with-lease` is `--force` with a seatbelt: it refuses the push if the remote moved since you last fetched, so you can't silently clobber a teammate. **Rule of thumb: never plain `--force` to a shared branch.**

### Shells we'll live in: PowerShell, cmd, Bash
Three different shells show up in this project, each speaks its own dialect.
- **cmd.exe** (Windows Command Prompt, prompt `C:\...>`): 1987 design, text-only, tiny command set. Avoid — only used for old `.bat` scripts.
- **PowerShell** (prompt `PS C:\...>`): modern Windows shell. Pipes **objects** (not text) between commands, full programming language, has Linux-style aliases (`ls`, `cd`, `cat`, `pwd`). This is where we work on the laptop.
- **Bash/sh** (inside Docker containers and GitHub Actions): standard Linux shell. Different syntax — `ls -la`, `export VAR=x`, `if [ -f file ]; then ... fi`. We learn it as it comes up.

The cmdlet `New-Item` is PowerShell-only and errors in cmd.exe — that's how we caught the wrong shell. Going forward: always launch PowerShell for CoopHub work.

---

## Phase 1b — Postgres + Redis in Docker Compose

### Image vs Container vs Volume vs Network
Four words to lock in.
- **Image** = read-only recipe (e.g. `postgres:16-alpine`). Built in layers; layers cache.
- **Container** = a running instance of an image. Throwaway by design.
- **Volume** = persistent storage that outlives any single container. Your data lives here, not in the container.
- **Network** = a private virtual LAN compose creates for the stack. Containers find each other by **service name** (`postgres`, `redis`) inside it. Outside it, those names mean nothing.

### `docker compose up -d` — what actually happens
1. Reads `docker-compose.yml`.
2. Reads `.env` and substitutes `${VARIABLE}` placeholders.
3. Pulls images from Docker Hub (first run only; cached after).
4. Creates the compose network and any declared volumes.
5. Starts each container, injecting env vars at runtime.
6. Maps `host:container` ports so the laptop can reach the containers via `localhost`.
7. With `-d` (detached), it returns the shell while containers keep running in the background.

`docker compose down` stops and removes containers + network (volumes kept). `docker compose down -v` also wipes volumes — data gone.

### Healthchecks
A small probe Docker runs inside the container repeatedly to answer "are you actually ready?" not just "did the process start?" Postgres uses `pg_isready`; Redis uses `redis-cli ping`. Phase 1c's API will wait on these via `depends_on: condition: service_healthy` so it doesn't crash by trying to connect before the DB accepts queries.

### `.env` vs `.env.example` vs `.gitignore`
- `.env.example` — template with placeholder values. **Committed.**
- `.env` — real values. **Gitignored, never committed.**
- `.gitignore` — has `.env` in its list (set in Phase 0).
Standard convention across modern projects. Stops credentials from leaking into git history (which is forever).

### `$$` escaping in compose YAML
A single `${VAR}` is substituted by compose before sending the YAML to Docker. A double `$${VAR}` escapes that substitution so the shell **inside the container** does it instead. Healthchecks run inside the container, so we use `$${POSTGRES_USER}` there.

### Declarative infrastructure
You describe **what you want** (a Postgres service, a Redis service, this network, this volume), the tool figures out **how**. Same idea powers Kubernetes manifests (Phase 4), Terraform (Phase 3), and ArgoCD (Phase 5). Imperative shell scripts ("install this, then start that") are the old way.
