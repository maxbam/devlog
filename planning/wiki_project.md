# Wiki / Portfolio Project

Living document. Updated as we go — decisions, progress, and what's next.

---

## Goal

A documentation site hosted on GitHub Pages that serves two audiences:

- **Me** — a searchable lookup for things I've figured out, so I don't re-solve the same problem.
- **Others** — evidence of how I work as a developer.

The site is also the first practice project. Building and deploying it is a CI/CD exercise, and writing it up is the site's first article.

The main portfolio project that follows (Phase 6) is a **pseudo-MMO prototype**: a Godot client with a Python backend that moves step by step to Rust. Target audience for the portfolio: **backend and general software engineering roles**. Details in [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).

---

## Decisions made

| Decision | Choice | Why |
|---|---|---|
| Docs platform | MkDocs Material (not GitHub Wiki) | Wiki has no build pipeline, poor theming, bad discoverability. MkDocs is Python, matches the rest of the stack. |
| Language | Python | Already familiar, used at work. |
| Environment | WSL | Containers are Linux; CI runs `ubuntu-latest`. Code lives in WSL filesystem, not `/mnt/c/`. |
| Deploy | GitHub Actions → Pages | Not `mkdocs gh-deploy` — that builds from local state and skips CI entirely. |
| Repo layout | One repo per project + one docs repo | Projects get thorough READMEs; docs site links out and holds cross-cutting material. |
| Stay on Material vs. migrate to Zensical | ~~Stay on Material for MkDocs 9.7.7 for now~~ → **Migrate to Zensical before Phase 4 content** (revised 2026-10-04) | Original call (2026-08-30): stay, because Zensical lacked the `blog` plugin. Revisit condition met: `blog` shipped in Zensical 0.0.64, `tags` in 0.0.58. Material's security-fix window ends Nov 2026. Migration cost is lowest now, with one page of content. |
| Repo name | `devlog` → `maxbam.github.io/devlog/` | Keeps the root `maxbam.github.io` free for a landing page later. Avoids `wiki/wiki` URL collision with GitHub's built-in wiki feature. |
| Phase 6 project | Pseudo-MMO prototype (40k-*style*, original IP): shared hub + instanced zones, 3 classes, one starting zone (decided 2026-10-13) | Big enough to need real backend engineering (API, database, real-time server, contracts), small enough to finish. Expandable zone by zone. Absorbs the original "small API + Postgres" plan as its meta backend. |
| Language path | Python first, then Rust — module by module, then service by service (decided 2026-10-13) | Start productive in a known language; move to Rust where it measurably pays off. Each migration becomes a devlog article with before/after numbers. Rust is learned by hand, not delegated. |
| AI role | Coach first, delegate later (decided 2026-10-16) | Learning by doing. Delegate and automate only once the topics are understood. Automation becomes its own article after Phase 6. |

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
Nav structure, `notes/` vs `projects/` split, blog plugin for notes. First article: setting up this site.
**Before starting:** ~~re-check Zensical's plugin compatibility page~~ Checked 2026-10-04: `blog` has landed. Migrate first (steps 4.1–4.2), then write.
**Steps:**
- 4.0 — ✅ Verified 2026-10-16 via push run 38911805234 (build + deploy green after Node 20 removal).
- 4.1 — ✅ Done 2026-10-16 on branch `feat/zensical-migration` (PR check proves it).
- 4.2 — Merge, confirm deploy.
- 4.3 — Nav structure, `notes/` + `projects/`, blog config.
- 4.4 — First article (written by me, not delegated).
**Checkpoint:** One real article published end to end.

### Phase 5 — Learning with AI as a coach
Claude Code in VS Code as a mentor, not a doer: `CLAUDE.md` mentor-mode conventions (guiding question → hint → concept → solution only on request), learning to investigate and debug, and when a handoff is still fine. Raw material: `planning/coaching_log.md`.
**Done early:** VS Code and Claude Code installed (confirmed 2026-10-04). Mentor mode active since 2026-10-16 (PR #8). Steps up to 4.1 were delegated, which gives a "before" picture to compare against.
**Checkpoint:** Article two — how to use AI to learn, with AI as a coach.

### Phase 6 — Pseudo-MMO prototype (main portfolio project)
~~Small API + Postgres, migrations, docker-compose for local dev, tests in CI.~~ Expanded 2026-10-13: the original backend project becomes the **meta backend** (subphase 6.1) of a pseudo-MMO prototype.
**Stack:** Godot 4 client (GDScript) · FastAPI + PostgreSQL + Redis meta backend · Python real-time game server, migrating to Rust · Docker Compose · GitHub Actions.
**Subphases:** 6.0 project setup → 6.1 meta backend → 6.2 networked skeleton → 6.3 combat core → 6.4 starting zone → 6.5 hub → 6.6 first Rust module (PyO3) → 6.7 Rust game server.
**Details:** [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md). One file per subphase as each starts.
**Checkpoint:** Two players level through the starting zone together; green CI; every major change written up in `projects/`.

### Phase 7 — AI automation in a larger project (after Phase 6)
Once the topics are understood: which tasks in a big software project to delegate or automate with AI, and how. Raw material: `planning/delegation_log.md`.
**Checkpoint:** Its own article.

---

## Risks

### MkDocs ecosystem fork (identified 2026-08-30)

The `mkdocs serve` output prints a warning about MkDocs 2.0. Background:

- MkDocs upstream has been effectively unmaintained since ~Aug 2024.
- MkDocs 2.0 removes the plugin system and rewrites theming, with no migration path. Material for MkDocs cannot work under it.
- Material for MkDocs entered **maintenance mode** (Nov 2025): critical bug/security fixes for at least 12 months, no new features. All former Insiders features are now free.
- The Material team's successor project is **Zensical** (Rust + Python, MIT, actively developed). Community forks also exist: ProperDocs, MaterialX.

**Why this is not urgent for us:** Material pins `mkdocs<2`, and our `requirements.txt` is fully pinned. The build cannot silently break.

**Blocker for migrating now:** ~~Zensical's `blog` plugin support is Tier 2 and not yet shipped. `tags` is also outstanding.~~ **Resolved (checked 2026-10-04):** `blog` supported since Zensical 0.0.64 (archives, categories, author profiles, pagination; only difference is no custom Python callables). `tags` supported since 0.0.58, including listings. Current Zensical version 0.0.67; 0.1.0 announced for 2026-11-05 — treat as a routine bump, not a reason to wait.

**Support window:** Material's critical bug/security fixes run until November 2026.

**Do not** set `NO_MKDOCS_2_WARNING=1` yet — the warning is useful signal, including in CI.

**Revisit:** start of Phase 4, and again if Material's support window lapses.

**Resolved 2026-10-16:** migrated to Zensical. Material's support window no longer affects us.

**Watch:** Zensical [#966](https://github.com/zensical/zensical/issues/966) — `--strict` may miss broken links to blog posts by source path. Relevant at step 4.3.

**Upcoming:** Zensical 0.1.0 due 2026-11-05. Routine pinned bump.

### GitHub Actions Node 20 deprecation (identified 2026-08-30)

Build logs warned that `actions/checkout@v4`, `actions/setup-python@v5`, and `actions/upload-artifact@v4` target Node 20 and are being forced onto Node 24. GitHub flipped the default on 2026-06-02; Node 20 is removed from runners 2026-09-16.

**Fixed:** bumped to `actions/checkout@v5` and `actions/setup-python@v6`.

**Update 2026-10-04:** Node 20 removal date has passed. Workflow not exercised since 2026-08-30 — verify with a manual run (step 4.0).

~~**Not ours to fix:** `upload-artifact@v4` is called from inside `upload-pages-artifact@v3`, a composite action. Open upstream issue. Nothing breaks — the job already runs on Node 24.~~ **Resolved 2026-10-16:** `upload-pages-artifact@v5` bundles `upload-artifact@v7`. Bumped. (v4+ excludes dotfiles from the artifact; our `site/` has none.)

**Also found 2026-10-16:** `deploy-pages@v4` warned too (it targets Node 20). Bumped to v5.

**Lesson:** we don't control the whole dependency tree. Pinning our own dependencies is necessary but not sufficient. Consider Dependabot for the `github-actions` ecosystem at Phase 6.

**Article material:** this is the first write-up. "Chose a tool, ecosystem forked underneath me, here's how I decided" is more useful than a setup tutorial.

### Ubuntu 26 runner migration (identified 2026-10-16)

`ubuntu-latest` moves to Ubuntu 26 starting 2026-10-19. Pinned both jobs to `ubuntu-24.04` so the runner doesn't change under the Zensical migration.

**Revisit:** after step 4.2.

### Phase 6 scope creep (identified 2026-10-13)

An MMO-shaped project grows without limit. Mitigations: pseudo-MMO scope (hub + small instances, not a world), one checkpoint per subphase, content as data, and no new system until the current subphase's checkpoint works. Full list of project risks in [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).

---

## Status

**Current phase:** 4 — Structure and first article
**Current step:** 4.2 — Merge, confirm deploy.
**Next up after Phase 5:** Phase 6.0 — game project setup.

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
| 2026-08-30 | Phase 3 done. Ruleset on `main`: require PR, require `build` check, block force pushes, restrict deletions. |
| 2026-10-04 | Back after a month. VS Code + Claude Code installed in WSL. Zensical revisit triggered: `blog` and `tags` both shipped. Decision reversed — migrate before Phase 4 content. |
| 2026-10-12 | Explored a 40k-style MMORPG as the Phase 6 project. Scoped down to a pseudo-MMO (hub + instances). Reviewed AI 3D tooling (Blender MCP, Meshy/Tripo). |
| 2026-10-13 | Phase 6 decided: Godot client, Python backend, staged move to Rust. Compared Godot, Unity, Unreal, and Bevy; Bevy rejected for now. Plan written up in `phase6_pseudo_mmo.md`. |
| 2026-10-16 | Step 4.0 done. Ruleset bypass list confirmed empty. Run 38911805234 green after Node 20 removal. Remaining Node 20 warnings fixed by bumping `deploy-pages` to v5 and `upload-pages-artifact` to v5. Runner pinned to `ubuntu-24.04` ahead of the Ubuntu 26 switch (2026-10-19). |
| 2026-10-16 | Step 4.1: migrated to Zensical 0.0.71 (classic variant), still on `mkdocs.yml`. MkDocs + Material removed from `requirements.txt`. Strict verified locally: a broken link and a broken anchor both exit 1. CI runs `zensical build --clean --strict`. |
| 2026-10-16 | Switched to mentor mode (PR #8). Phase 5 refocused on learning with AI as a coach. AI automation moved to Phase 7, after Phase 6. |

---

## Open questions

- Custom domain, or stick with `username.github.io`? Deferred; easy to add later.
- Move config from `mkdocs.yml` to `zensical.toml`? (Not required; Zensical reads `mkdocs.yml` natively.)
- Switch theme variant from `classic` to `modern`?
- ~~Which backend project for Phase 6? Leaning toward a small API with a real database over a toy CRUD demo.~~ **Resolved 2026-10-13:** pseudo-MMO prototype; the API + database becomes its meta backend. Phase 6 open questions now live in [`phase6_pseudo_mmo.md`](phase6_pseudo_mmo.md).
