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
