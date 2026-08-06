# Toolchain decisions

Decision records for the docs *toolchain* (not the docs content). This file lives outside
`docs/` on purpose: everything under `docs/` is published to the public site and must have a
German sibling (CI enforces it). The README policy — "only the user-facing docs, never the
plugin's source, its tests, or its internal planning" — excludes *plugin* internals. A record
about this repo's own build toolchain is not plugin-internal, so it lives here, unpublished.

## 1. Stay on mkdocs-material 9.7.x past its end of life (deliberate freeze)

**Status: PROPOSED (draft).** This record is a recommendation. Merging the pull request that
adds it makes it the accepted record; the status line is not edited afterwards — merge is the
acceptance. Rejecting the PR rejects the proposal.

Date: 2026-08-06 · Relates to:
[issue #14](https://github.com/hilfor/kimai-jira-docs/issues/14) (this decision),
[issue #16](https://github.com/hilfor/kimai-jira-docs/issues/16) (constraints)

### Context

The site builds with `mkdocs-material` (pinned `>=9.7.7,<10`) and `mkdocs-static-i18n`
(the EN/DE split via `*.de.md` suffix files and the Material language switcher).

- **mkdocs-material is in maintenance mode and reaches end of life on 2026-11-05.** Until
  then it receives critical bug fixes and security updates only; after that date, minor
  issues may not receive a fix. Existing sites keep working.
  Source: <https://github.com/squidfunk/mkdocs-material/issues/8523>
- **MkDocs core (1.x) is frozen**, and MkDocs 2.0 is developed under a closed model, removes
  the plugin system, and offers no migration path. mkdocs-material 9.7.5+ pins `mkdocs<2`, so
  our stack cannot accidentally resolve to the incompatible 2.0.
  Source: <https://squidfunk.github.io/mkdocs-material/blog/2026/02/18/mkdocs-2.0/>
- **mkdocs-static-i18n is frozen.** The maintainer stated on 2026-03-01 that with MkDocs core
  frozen he will not invest more time in the plugin, and that no Zensical compatibility work
  will happen because Zensical plans native i18n. Last release: 1.3.1 (2026-02-20). The
  plugin still works with our pinned stack.
  Source: <https://github.com/ultrabug/mkdocs-static-i18n/issues/342#issuecomment-3980785833>

### Options

**(a) Migrate to Zensical** — the successor by the mkdocs-material team.

Not feasible today. The i18n question decides it, and the answer is no:

- Zensical is alpha software: version 0.0.53 (2026-08-04), PyPI classifier
  "Development Status :: 3 - Alpha". Source: <https://pypi.org/project/zensical/>
- Internationalization sits in the roadmap's "Next up" section — planned, not shipped, with
  explicitly no dates promised. Source: <https://zensical.org/about/roadmap/>
- `mkdocs-static-i18n` support is a Tier-2 backlog commitment
  (<https://zensical.org/compatibility/plugins/>), and the tracking issue is open with no
  activity since 2025-11-13. Source: <https://github.com/zensical/backlog/issues/1>

Zensical cannot build our EN/DE site today. The good news for later: Zensical intends to
adopt this plugin's configuration structure, so the eventual migration is designed to be
cheap (<https://fosstodon.org/@squidfunk/116153447850371721>, confirmed by the plugin
maintainer in the issue linked above).

**(b) Switch to another MkDocs theme** — rejected.

Every MkDocs 1.x theme sits on the same frozen MkDocs core and the same frozen i18n plugin,
so this spends migration effort without leaving the end-of-life platform. Worse, our language
switcher comes from `reconfigure_material: true`, which is Material-specific — another theme
loses it. No specific alternative theme was evaluated in depth because (a) is blocked only
temporarily and (c) carries low risk (below).

**(c) Stay frozen on 9.7.x deliberately — RECOMMENDED.**

- Security and critical fixes for the two pinned packages still flow until 2026-11-05: the
  open ranges (`>=9.7.7,<10`, `>=1.3.1,<2`) accept them, and the weekly Dependabot pip
  updates cover exactly these two manifest entries — nothing else is in the manifest.
- Transitive deps (jinja2, markdown, pygments, ...) are maintained independently and stay
  fresh through a different mechanism: there is no lockfile, so the
  `pip install -r requirements.txt` in `deploy.yml` resolves the newest allowed versions on
  every build. Dependabot does not see them and its alerts do not fire for them.
- **Risk window after 2026-11-05, build side:** unpatched defects in `mkdocs`,
  `mkdocs-material`, and `mkdocs-static-i18n`. Contained: the toolchain runs only at build
  time in GitHub Actions on our own trusted Markdown — the published artifact is static HTML
  on GitHub Pages, with no server-side code at runtime.
- **Risk window after 2026-11-05, client side:** Material ships JavaScript into every
  visitor's browser (the search UI via `reconfigure_search: true`, plus theme code), and
  every build re-ships it unchanged. This defect class is real: 9.7.7 (2026-07-17) fixed a
  DOM-based XSS in search suggestions
  (<https://github.com/squidfunk/mkdocs-material/blob/master/CHANGELOG>). After EOL, the
  next such defect gets no fix. Bound: the site serves no user content and the Pages origin
  has no auth or session, so an attack needs a crafted URL and gains almost nothing.
- `mkdocs<2` (pinned by mkdocs-material 9.7.5+) protects the freeze from the incompatible
  MkDocs 2.0.

### Constraints any future toolchain must keep (verified facts from [issue #16](https://github.com/hilfor/kimai-jira-docs/issues/16))

- `docs/googlef0456d0375538b5c.html` is the Google Search Console verification artifact. The
  toolchain must keep copying it **verbatim into the site root**; removing it drops the
  verification.
- The toolchain must keep generating `sitemap.xml`. Today it is auto-generated because
  `site_url` is set: 24 URLs, each with EN/DE hreflang alternates.

### Re-evaluation trigger and migration steps (proposed follow-up)

When the PR that adds this record merges, **file a re-evaluation issue in this repo** — the
trigger needs an owner and a date, not a watched upstream issue. That issue tracks:

- **Trigger:** the Zensical roadmap moves internationalization out of "Next up" into shipped,
  with suffix-based content organization (<https://zensical.org/about/roadmap/>), and
  Zensical leaves alpha (<https://pypi.org/project/zensical/>). Do not anchor on
  <https://github.com/zensical/backlog/issues/1>: Zensical can ship native i18n without ever
  closing that plugin-compatibility issue.
- **Date backstop:** re-check the roadmap on the EOL date (2026-11-05), then every three
  months until the migration happens.
- **EOL hardening:** at the 2026-11-05 check, pin the full resolved dependency set
  (`pip freeze` into a constraints file). Builds become reproducible, and the transitives
  enter Dependabot's update surface (see the mechanism note under option (c)).

When the trigger fires, file one migration issue with:

1. `requirements.txt` — replace `mkdocs-material` + `mkdocs-static-i18n` with `zensical`.
2. `mkdocs.yml` — migrate the config (palette, features, i18n languages, `nav_translations`);
   verify the language switcher still works under the `/kimai-jira-docs/` subpath.
3. `.github/workflows/deploy.yml` — swap the build command, keep the German-sibling check,
   confirm the output path for the Pages artifact.
4. Verify both constraints above against the new build output, plus EN and DE page trees.
