# CLAUDE.md — devlog

Documentation site and portfolio for maxbam, published at `https://maxbam.github.io/devlog/`.
Plans and decisions live in `planning/`. Read `planning/wiki_project.md` before any non-trivial task; its **Status** section says what is being worked on.

## Stack

- MkDocs Material 9.7.7 with `strict: true`. **Migration to Zensical is planned (roadmap step 4.1).** Update this file when it lands.
- Python 3.12 in WSL. Dependencies are fully pinned in `requirements.txt`.
- Deploy: GitHub Actions → GitHub Pages (`.github/workflows/`). Two jobs: `build` → `deploy`. Pull requests build but never deploy.

## Layout

- `docs/` — site content. `notes/` and `projects/` sections are planned (step 4.3).
- `mkdocs.yml` — site config.
- `planning/` — living planning docs (`wiki_project.md`, `phase6_pseudo_mmo.md`). Not part of the site.
- `planning/delegation_log.md` — log of delegated tasks (see below).

## Commands

- `mkdocs serve` — local preview.
- `mkdocs build` — must exit 0. With `strict: true`, any warning (e.g. a broken link) fails the build, locally and in CI.

## Rules

- **Never commit or push to `main`.** The ruleset requires a PR and a green `build` check. Work on a branch; I open and merge the PR.
- **Never use `mkdocs gh-deploy`.** Deploys go through CI only.
- **Never weaken `strict: true`** or silence warnings to get a build green. Fix the cause.
- **Do not set `NO_MKDOCS_2_WARNING`.** The warning is intentional signal.
- **Pin exact versions** when adding or bumping a dependency, and say why in the commit message.
- **GitHub Actions:** keep minimal `permissions`, keep the deploy-job gate, and use current major versions (`actions/checkout@v5`, `actions/setup-python@v6`).
- Run `mkdocs build` before saying a task is done, and report the result.

## Content

- `notes/` is fast and informal, for me. Fixing formatting, links, and front matter there is fine.
- `projects/` is polished writing for readers. **I write all article prose.** Do not draft, rewrite, or "improve" article text. Fix typos, links, or formatting only when I ask.

## Delegate vs. not

- **Yours:** config, nav, the Zensical migration, workflow YAML, fixing `strict` errors, boilerplate, scripts.
- **Mine:** articles, decisions, anything listed as "written by hand" in `planning/`.
- When a task implies a decision (a new tool, a changed approach, a new dependency), propose options and stop. Don't decide.

## Planning docs

- When a task completes a roadmap step, propose the updates to **Status** and **Progress log** in `planning/wiki_project.md`, and apply them after I confirm.
- Never edit the **Decisions made** tables without asking.

## Delegation log

After each delegated task, append one row to `planning/delegation_log.md`:

`| date | task | what you did | files changed | my notes |`

Leave **my notes** empty. I fill it in with what I had to correct and how long it took. This is raw material for a Phase 5 article.
