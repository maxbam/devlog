# Wiki / Portfolio Project

Living document. Updated as we go — decisions, progress, and what's next.

---

## Goal

A documentation site hosted on GitHub Pages that serves two audiences:

- **Me** — a searchable lookup for things I've figured out, so I don't re-solve the same problem.
- **Others** — evidence of how I work as a developer.

The site is also the first practice project. Building and deploying it is a CI/CD exercise, and writing it up is the site's first article.

---

## Decisions made

| Decision | Choice | Why |
|---|---|---|
| Docs platform | MkDocs Material (not GitHub Wiki) | Wiki has no build pipeline, poor theming, bad discoverability. MkDocs is Python, matches the rest of the stack. |
| Language | Python | Already familiar, used at work. |
| Environment | WSL | Containers are Linux; CI runs `ubuntu-latest`. Code lives in WSL filesystem, not `/mnt/c/`. |
| Deploy | GitHub Actions → Pages | Not `mkdocs gh-deploy` — that builds from local state and skips CI entirely. |
| Repo layout | One repo per project + one docs repo | Projects get thorough READMEs; docs site links out and holds cross-cutting material. |
| Stay on Material vs. migrate to Zensical | Stay on Material for MkDocs 9.7.7 **for now** | Ecosystem forked Feb 2026 (see Risks). Material is pinned to `mkdocs<2` so nothing breaks. Zensical does not yet support the `blog` plugin, which Phase 4 needs. Migration is designed to be easy later — Zensical reads `mkdocs.yml` natively. |
| Repo name | `devlog` → `maxbam.github.io/devlog/` | Keeps the root `maxbam.github.io` free for a landing page later. Avoids `wiki/wiki` URL collision with GitHub's built-in wiki feature. |

### Guiding principles

- **Practices are answers to problems.** Build something slightly too simple, feel the pain, then document why it changed. More honest and more interesting than a checklist.
- **Two content areas, different standards.** `notes/` is fast and messy and for me. `projects/` is polished and for readers. Making everything presentable is how documentation habits die.
- **Timebox the site.** A day, not a weekend. Empty docs sites stay empty.

---

## Roadmap

Phases are sequential. Each has a checkpoint — something that visibly works — before moving on.

### Phase 1 — Environment baseline ✅
Confirm WSL, Python, git, and SSH are set up correctly before writing any code.
**Checkpoint:** `ssh -T git@github.com` authenticates, `python3 --version` is 3.11+.
**Verified:** Python 3.12.3, git 2.55.0, bash, working dir `/home/max/projects/github` (inside WSL filesystem), SSH auth as `maxbam`.

### Phase 2 — Repo and local site
Create the repo, venv, pinned dependencies, minimal `mkdocs.yml` with `strict: true`.
**Checkpoint:** `mkdocs serve` renders a homepage locally.

### Phase 3 — CI/CD deploy ✅
GitHub Actions workflow: build on push to `main`, deploy via the Pages action. Branch protection on `main`.
**Checkpoint:** A push to `main` puts the site live on `github.io`. A broken link fails the build.
**Built:** Two-job workflow (`build` → `deploy`) with explicit minimal `permissions`, `concurrency: pages`, and OIDC deploy via `actions/deploy-pages`. Pages source set to *GitHub Actions*, not a branch.
**Triggers:** `push` to `main`, `pull_request` to `main`, `workflow_dispatch`. Deploy job gated with `if: github.event_name == 'push' && github.ref == 'refs/heads/main'` so PRs validate but never publish.
**Verified:** `strict: true` rejects a link to a non-existent page — reproduced locally (`mkdocs build`, non-zero exit) and on a PR, with `main` untouched.

### Phase 4 — Structure and first article
Nav structure, `notes/` vs `projects/` split, blog plugin for notes. First article: setting up this site.
**Before starting:** re-check Zensical's plugin compatibility page. If `blog` has landed, migrate *before* writing content.
**Checkpoint:** One real article published end to end.

### Phase 5 — Claude Code + VS Code workflow
Install in WSL, `CLAUDE.md` conventions, what to delegate and what not to.
**Checkpoint:** A documented workflow, written up as article two.

### Phase 6 — First backend project
Small API + Postgres, migrations, docker-compose for local dev, tests in CI.
**Checkpoint:** Green CI, and the project documented in `projects/`.

---

## Risks

### MkDocs ecosystem fork (identified 2026-08-30)

The `mkdocs serve` output prints a warning about MkDocs 2.0. Background:

- MkDocs upstream has been effectively unmaintained since ~Aug 2024.
- MkDocs 2.0 removes the plugin system and rewrites theming, with no migration path. Material for MkDocs cannot work under it.
- Material for MkDocs entered **maintenance mode** (Nov 2025): critical bug/security fixes for at least 12 months, no new features. All former Insiders features are now free.
- The Material team's successor project is **Zensical** (Rust + Python, MIT, actively developed). Community forks also exist: ProperDocs, MaterialX.

**Why this is not urgent for us:** Material pins `mkdocs<2`, and our `requirements.txt` is fully pinned. The build cannot silently break.

**Blocker for migrating now:** Zensical's `blog` plugin support is Tier 2 and not yet shipped. `tags` is also outstanding. Phase 4 depends on both.

**Do not** set `NO_MKDOCS_2_WARNING=1` yet — the warning is useful signal, including in CI.

**Revisit:** start of Phase 4, and again if Material's support window lapses.

### GitHub Actions Node 20 deprecation (identified 2026-08-30)

Build logs warned that `actions/checkout@v4`, `actions/setup-python@v5`, and `actions/upload-artifact@v4` target Node 20 and are being forced onto Node 24. GitHub flipped the default on 2026-06-02; Node 20 is removed from runners 2026-09-16.

**Fixed:** bumped to `actions/checkout@v5` and `actions/setup-python@v6`.

**Not ours to fix:** `upload-artifact@v4` is called from inside `upload-pages-artifact@v3`, a composite action. Open upstream issue. Nothing breaks — the job already runs on Node 24.

**Lesson:** we don't control the whole dependency tree. Pinning our own dependencies is necessary but not sufficient. Consider Dependabot for the `github-actions` ecosystem at Phase 6.

**Article material:** this is the first write-up. "Chose a tool, ecosystem forked underneath me, here's how I decided" is more useful than a setup tutorial.

---

## Status

**Current phase:** 3 — CI/CD deploy (closing)
**Current step:** 3.4 — Branch protection ruleset on `main`, then Phase 4

---

## Progress log

| Date | What happened |
|---|---|
| 2026-08-30 | Project scoped. Platform, language, environment, and deploy decisions locked in. Roadmap drafted. |
| 2026-08-30 | Phase 1 done. Toolchain verified, git identity configured. Repo named `devlog`. |
| 2026-08-30 | Repo cloned. venv + `mkdocs-material` 9.7.7 installed, `requirements.txt` pinned. Minimal `mkdocs.yml` + `docs/index.md` created; local build succeeds. |
| 2026-08-30 | Hit the MkDocs 2.0 warning. Researched the ecosystem fork; decided to stay on Material and revisit at Phase 4. Logged under Risks. |
| 2026-08-30 | Phase 2 done. First commit pushed; site skeleton on `main`. |
| 2026-08-30 | Deploy workflow added; site live. Bumped actions for Node 24. Added `pull_request` trigger and gated the deploy job so PRs validate without publishing. Confirmed `strict: true` fails the build on a broken link, and that merging a PR redeploys. |

---

## Open questions

- Custom domain, or stick with `username.github.io`? Deferred; easy to add later.
- Which backend project for Phase 6? Leaning toward a small API with a real database over a toy CRUD demo.
