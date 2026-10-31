# Wiki / Portfolio Project

Living document: the plan, decisions, and progress. Tasks and what's next live on the TickTick board, not here.

---

## Goal

A documentation site hosted on GitHub Pages that serves two audiences:

- **Me** — a searchable lookup for things I've figured out, so I don't re-solve the same problem.
- **Others** — evidence of how I work as a developer.

The site is also the first practice project. Building and deploying it is a CI/CD exercise.

The main portfolio project that follows (Phase 6) is a **pseudo-MMO prototype**: a Godot client with a Python backend that moves step by step to Rust. Target audience for the portfolio: **backend and general software engineering roles**. Details in [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).

A separate landing page at `maxbam.io` will link out to the devlog and the other projects. Details in [`landing_page.md`](landing_page.md).

---

## Decisions made

| Decision | Choice | Why |
|---|---|---|
| Docs platform | Zensical (modern variant), config in `zensical.toml` (not GitHub Wiki) | Wiki has no build pipeline, poor theming, bad discoverability. Zensical is the successor to Material for MkDocs (maintenance mode), Python-installable, and supports `blog` and `tags`. Background under Risks. |
| Language | Python | Already familiar, used at work. |
| Environment | WSL | Containers are Linux; CI runs on Ubuntu. Code lives in WSL filesystem, not `/mnt/c/`. |
| Deploy | GitHub Actions → Pages | Never from a local machine — that builds from local state and skips CI entirely. |
| Repo layout | One repo per project + one docs repo | Projects get thorough READMEs; docs site links out and holds cross-cutting material. |
| Repo name | `devlog` | Avoids `wiki/wiki` URL collision with GitHub's built-in wiki feature. |
| Domain | `devlog.maxbam.io`; `maxbam.io` redirects there until the landing page exists | The devlog is the technical write-up; the landing page links out to projects, which may have their own domains. A subdomain is cheap to change later; moving a live site isn't. |
| Phase 6 project | Pseudo-MMO prototype (40k-*style*, original IP): shared hub + instanced zones, 3 classes, one starting zone | Big enough to need real backend engineering (API, database, real-time server, contracts), small enough to finish. Expandable zone by zone. Absorbs the original "small API + Postgres" plan as its meta backend. |
| Language path | Python first, then Rust — module by module, then service by service | Start productive in a known language; move to Rust where it measurably pays off. Each migration becomes a devlog article with before/after numbers. Rust is learned by hand, not delegated. |
| AI role | Coach first, delegate later | Learning by doing. Delegate and automate only once the topics are understood. Automation becomes its own article after Phase 6. |
| Task tracking | TickTick GTD board, not the repo | One owner per fact: planning docs hold the plan, decisions, and history; TickTick holds what to do now. Avoids stale status sections. |

### Guiding principles

- **Practices are answers to problems.** Build something slightly too simple, feel the pain, then document why it changed. More honest and more interesting than a checklist.
- **Two content areas, one standard.** `blog/` holds self-contained articles, dated and tagged. `projects/` holds one area per long-running project: its documentation and, later, a short public roadmap. Both are public, so both get a minimum polish. Rough thinking stays off the site: to-dos in TickTick, planning in `planning/`. Making everything presentable by hand is how documentation habits die, so the polish is shared: the idea and the draft come from me, and AI helps make them presentable.
- **Timebox the site.** A day, not a weekend. Empty docs sites stay empty.

---

## Roadmap

Phases are sequential. Each has a checkpoint — something that visibly works — before moving on.

### Phase 1 — Environment baseline ✅
Confirm WSL, Python, git, and SSH are set up correctly before writing any code.
**Checkpoint:** `ssh -T git@github.com` authenticates, `python3 --version` is 3.11+.
**Verified:** Python 3.12.3, git 2.55.0, bash, working dir `/home/max/projects/github` (inside WSL filesystem), SSH auth as `maxbam`.

### Phase 2 — Repo and local site ✅
Create the repo, venv, pinned dependencies, minimal `mkdocs.yml` with `strict: true`.
**Checkpoint:** `mkdocs serve` renders a homepage locally.

### Phase 3 — CI/CD deploy ✅
GitHub Actions workflow: build on push to `main`, deploy via the Pages action. Branch protection on `main`.
**Checkpoint:** A push to `main` puts the site live on `github.io`. A broken link fails the build.
**Built:** Two-job workflow (`build` → `deploy`) with explicit minimal `permissions`, `concurrency: pages`, and OIDC deploy via `actions/deploy-pages`. Pages source set to *GitHub Actions*, not a branch.
**Triggers:** `push` to `main`, `pull_request` to `main`, `workflow_dispatch`. Deploy job gated with `if: github.event_name == 'push'` so PRs validate but never publish. (`push` only fires on `main`, so no separate branch check is needed.)
**Verified:** `strict: true` rejects a link to a non-existent page — reproduced locally (`mkdocs build`, non-zero exit) and on a PR, with `main` untouched.

### Phase 4 — Structure and first article
Nav structure, `blog/` vs `projects/` split, blog plugin for `blog/`. First article: learning with AI as a coach (the Phase 5 article, pulled forward).
**Steps:**
- 4.0 — ✅ Verified 2026-10-16 via push run 38911805234 (build + deploy green after Node 20 removal).
- 4.1 — ✅ Done 2026-10-16 on branch `feat/zensical-migration` (PR check proves it).
- 4.2 — ✅ Merge, confirm deploy. Done 2026-10-16: PR #7 merged, push run 38921045348 green.
- 4.3 — ✅ Nav structure, `blog/` + `projects/`, blog config. Done 2026-10-22: PR #14, run 39848497254 green.
- 4.4 — First article: learning with AI as a coach (written by me, not delegated). Also meets the Phase 5 checkpoint.
**Checkpoint:** One real article published end to end.

### Phase 5 — Learning with AI as a coach
Claude Code in VS Code as a mentor, not a doer: `CLAUDE.md` mentor-mode conventions (guiding question → hint → concept → solution only on request), learning to investigate and debug, and when a handoff is still fine. Raw material: `coached` rows in the progress logs.
**Done early:** VS Code and Claude Code installed (confirmed 2026-10-04). Mentor mode active since 2026-10-16 (PR #8). Steps up to 4.1 were delegated, which gives a "before" picture to compare against.
**Checkpoint:** The first article (step 4.4) — how to use AI to learn, with AI as a coach.

### Phase 6 — Pseudo-MMO prototype (main portfolio project)
~~Small API + Postgres, migrations, docker-compose for local dev, tests in CI.~~ Expanded 2026-10-13: the original backend project becomes the **meta backend** (subphase 6.1) of a pseudo-MMO prototype.
**Stack:** Godot 4 client (GDScript) · FastAPI + PostgreSQL + Redis meta backend · Python real-time game server, migrating to Rust · Docker Compose · GitHub Actions.
**Subphases:** 6.0 project setup → 6.1 meta backend → 6.2 networked skeleton → 6.3 combat core → 6.4 starting zone → 6.5 hub → 6.6 first Rust module (PyO3) → 6.7 Rust game server.
**Details:** [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).
**Checkpoint:** Two players level through the starting zone together; green CI; every major change documented in `projects/pseudo-mmo/`; self-contained write-ups as blog posts.

### Phase 7 — AI automation in a larger project (after Phase 6)
Once the topics are understood: which tasks in a big software project to delegate or automate with AI, and how. Raw material: `delegated` rows in the progress logs.
**Checkpoint:** Its own article.

---

## Risks

### Active

#### Phase 6 scope creep (identified 2026-10-13)

An MMO-shaped project grows without limit. Mitigations: pseudo-MMO scope (hub + small instances, not a world), one checkpoint per subphase, content as data, and no new system until the current subphase's checkpoint works. Full list of project risks in [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).

### Resolved

#### MkDocs ecosystem fork (identified 2026-08-30)

The `mkdocs serve` output prints a warning about MkDocs 2.0. Background:

- MkDocs upstream has been effectively unmaintained since ~Aug 2024.
- MkDocs 2.0 removes the plugin system and rewrites theming, with no migration path. Material for MkDocs cannot work under it.
- Material for MkDocs entered **maintenance mode** (Nov 2025): critical bug/security fixes for at least 12 months, no new features. All former Insiders features are now free.
- The Material team's successor project is **Zensical** (Rust + Python, MIT, actively developed). Community forks also exist: ProperDocs, MaterialX.

**Why this is not urgent for us:** ~~Material pins `mkdocs<2`, and our `requirements.txt` is fully pinned. The build cannot silently break.~~ **Resolved (2026-10-20):** mkdocs dependency is removed because of switch to zensical

**Blocker for migrating now:** ~~Zensical's `blog` plugin support is Tier 2 and not yet shipped. `tags` is also outstanding.~~ **Resolved (checked 2026-10-04):** `blog` supported since Zensical 0.0.64 (archives, categories, author profiles, pagination; only difference is no custom Python callables). `tags` supported since 0.0.58, including listings. Current Zensical version at the time: 0.0.67.

**Support window:** Material's critical bug/security fixes run until November 2026.

**Resolved 2026-10-16:** migrated to Zensical. Material's support window no longer affects us.

#### GitHub Actions Node 20 deprecation (identified 2026-08-30)

Build logs warned that `actions/checkout@v4`, `actions/setup-python@v5`, and `actions/upload-artifact@v4` target Node 20 and are being forced onto Node 24. GitHub flipped the default on 2026-06-02; Node 20 is removed from runners 2026-09-16.

**Fixed:** bumped to `actions/checkout@v5` and `actions/setup-python@v6`.

**Update 2026-10-04:** Node 20 removal date has passed. Workflow not exercised since 2026-08-30 — verify with a manual run (step 4.0).

**Update 2026-10-21:** bumped to `actions/checkout@v7` and `actions/setup-python@v7`.

~~**Not ours to fix:** `upload-artifact@v4` is called from inside `upload-pages-artifact@v3`, a composite action. Open upstream issue. Nothing breaks — the job already runs on Node 24.~~ **Resolved 2026-10-16:** `upload-pages-artifact@v5` bundles `upload-artifact@v7`. Bumped. (v4+ excludes dotfiles from the artifact; our `site/` has none.)

**Also found 2026-10-16:** `deploy-pages@v4` warned too (it targets Node 20). Bumped to v5.

**Lesson:** we don't control the whole dependency tree. Pinning our own dependencies is necessary but not sufficient.

**Article material:** this is the first write-up. "Chose a tool, ecosystem forked underneath me, here's how I decided" is more useful than a setup tutorial.

#### Ubuntu 26 runner migration (identified 2026-10-16)

`ubuntu-latest` moves to Ubuntu 26 starting 2026-10-19. Pinned both jobs to `ubuntu-24.04` so the runner doesn't change under the Zensical migration.

**Update 2026-10-21:** bumped to `ubuntu-26.04`.

---

## Progress log

Mode: `coached` (did it myself with AI guidance), `delegated` (handed off to AI), `solo`, or `—` (before modes were tracked).

| Date | Mode | What happened |
|---|---|---|
| 2026-08-30 | — | Project scoped. Platform, language, environment, and deploy decisions locked in. Roadmap drafted. |
| 2026-08-30 | — | Phase 1 done. Toolchain verified, git identity configured. Repo named `devlog`. |
| 2026-08-30 | — | Repo cloned. venv + `mkdocs-material` 9.7.7 installed, `requirements.txt` pinned. Minimal `mkdocs.yml` + `docs/index.md` created; local build succeeds. |
| 2026-08-30 | — | Hit the MkDocs 2.0 warning. Researched the ecosystem fork; decided to stay on Material and revisit at Phase 4. Logged under Risks. |
| 2026-08-30 | — | Phase 2 done. First commit pushed; site skeleton on `main`. |
| 2026-08-30 | — | Deploy workflow added; site live. Actions bumped for Node 24. Deploy job gated so PRs build without publishing. Verified that strict fails on a broken link and that merging a PR redeploys. |
| 2026-08-30 | — | Phase 3 done. Ruleset on `main`: require PR, require `build` check, block force pushes, restrict deletions. |
| 2026-10-04 | — | Back after a month. VS Code + Claude Code installed in WSL. Zensical revisit triggered: `blog` and `tags` both shipped. Decision reversed — migrate before Phase 4 content. |
| 2026-10-16 | delegated | Step 4.0 done. Ruleset bypass list confirmed empty. Remaining Node 20 warnings fixed (`deploy-pages` and `upload-pages-artifact` → v5); runner pinned to `ubuntu-24.04`. **Lesson:** pinning our own dependencies isn't enough. |
| 2026-10-16 | delegated | Step 4.1: migrated to Zensical 0.0.71 (classic variant, still on `mkdocs.yml`); MkDocs + Material removed. Strict verified: a broken link and a broken anchor both exit 1. |
| 2026-10-16 | solo | Step 4.2 done (PR #7); build + deploy green. |
| 2026-10-16 | delegated | Switched to mentor mode (PR #8). |
| 2026-10-17 | coached | Phase 5 refocused on learning with AI as a coach; AI automation moved to Phase 7. Coaching log set up (PR #9). **Lesson:** when a decision changes, check other files for stale references (first question). |
| 2026-10-18 | delegated | Planning docs overhauled: tasks and open questions moved to TickTick, Status section removed, Decisions tables cut to the current choice, separate logs merged into this one. **Lesson:** one source of truth per fact, not per item. |
| 2026-10-20 | coached | Config moved to `zensical.toml`; theme switched to modern. Local strict proven with a deliberately broken link. **Lesson:** CI only sees the exit code (hint). |
| 2026-10-21 | solo | Bumped `checkout` and `setup-python` to v7 and runners to `ubuntu-26.04`; CLAUDE.md updated (PR #12). Runner stays pinned, not `ubuntu-latest`, so OS changes land as a PR. |
| 2026-10-22 | coached | Content model redefined: `notes/` → `blog/` (dated, tagged articles); `projects/` holds one area per long-running project, both to one public standard. CLAUDE.md: idea and draft are mine; AI edits text only on request. **Lesson:** decide what a section holds before naming it (concept). |
| 2026-10-22 | delegated | Checked Zensical #966 on 0.0.71 in a scratch copy: links to posts by source file resolve to the dated URL, and strict fails on a missing post. **Lesson:** link to posts by source file, not URL. |
| 2026-10-22 | coached | Step 4.3 done (PR #14). Blog plugin on defaults, placeholder project page, automatic nav, blog index with intro. **Lesson:** `serve` isn't what ships; check `build` (first question). |
| 2026-10-24 | delegated | Progress log shortened: run IDs and verification details dropped, duplicate Phase 6 rows removed, lessons cut to the essence. **Lesson:** details belong in the PR, not the log. |
| 2026-10-24 | coached | Custom domain: devlog.maxbam.io via CNAME (DNS only), maxbam.io verified on the account, HTTPS enforced, site_url and CLAUDE.md updated. **Lesson:** DNS knows hostnames, not paths (concept). |
| 2026-10-25 | coached | maxbam.io and www.maxbam.io redirect (302) to devlog.maxbam.io via a Cloudflare redirect rule; proxied placeholder A record (192.0.2.1), www as CNAME. **Lesson:** "doesn't exist" gets cached too (concept). |
