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

**This repo is the single source of truth for the user-facing documentation.** The per-feature
guides (`docs/features/`), screenshots (`docs/img/`), install/configure pages, and the EN/DE
translations all live here — edit them here.

The plugin repo deliberately keeps **no** `docs/features` or `docs/img` (that duplication is what
used to drift), and its own `docs/` tree is `export-ignore`d, so the plugin ZIP ships only
`README.md` + `LICENSE` (plus the runtime code). The plugin README is a summary + quick start that
links here for the deep guides and references this site's images by URL. See the plugin repo's
`docs/dev/docs.md` for the same policy from the other side.

## Local preview

```bash
pip install -r requirements.txt
mkdocs serve   # http://127.0.0.1:8000
```

## Publish

Enable **Settings → Pages → Source: GitHub Actions** once; every push to `main` then rebuilds and
deploys automatically.
