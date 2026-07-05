# kimai-jira-docs

Public documentation site for the **JiraBundle** Kimai plugin (Jira worklog sync). Published to
GitHub Pages via MkDocs Material — free, because this repo is public.

**This repo contains only the user-facing docs** — never the plugin source, tests, or internal
planning. The plugin itself lives in a separate private repository; that is what "features
visible, internals hidden" means here.

## Contents

- `docs/index.md` — landing page (what it does, install, configure).
- `docs/features/*.md` — one guide per capability.
- `docs/img/*.png` — screenshots.
- `mkdocs.yml` — site config; `.github/workflows/deploy.yml` — build + Pages deploy.

## Source of truth

`docs/features/` and `docs/img/` are the **same files that ship inside the plugin ZIP**, kept in
the private plugin repo. They are synced here on each plugin release (README-to-`index.md` links
are rewritten from `../../README.md` to `../index.md` for the site). Edit docs in the plugin repo,
not here, to avoid drift.

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve   # http://127.0.0.1:8000
```

## Publish

Enable **Settings → Pages → Source: GitHub Actions** once; every push to `main` then rebuilds and
deploys automatically.
