# CLAUDE.md

Guidance for Claude Code when working in this repo. This is a **personal research website** built on `al-folio` v1.x — a thin Jekyll starter that pulls runtime from versioned sibling gems (`al_folio_core`, `al_search`, `al_math`, etc.). You won't normally need to touch those gems; you're editing content, config, and the occasional local override.

@AGENTS.md

## Run the site (Docker only)

**Do not invoke `bundle`, `gem`, `jekyll`, or `bundle exec al-folio` on the host.** The host Ruby/Bundler may be missing or version-skewed. All Ruby-side commands run inside the container:

```bash
docker compose up -d                                              # serves at http://127.0.0.1:8080/research/
docker compose logs --tail=80 jekyll                              # build output / errors
docker compose restart jekyll                                     # if a _config.yml change didn't pick up
docker compose exec jekyll bundle exec jekyll build --baseurl /research
docker compose exec jekyll bundle exec al-folio upgrade audit
```
News
Teaching
People
Bookself


Host-side commands that are fine: `npm` scripts, `curl`, `git`, file edits.

## Things to know when editing this site

- **Plugin list lives in two places.** Adding/removing a plugin means editing **both** `Gemfile` (pinned version) and the `plugins:` list in `_config.yml`. They must stay in sync.
- **Features are gated twice.** A feature renders only when (a) its site-wide flag is on in `_config.yml` (e.g. `search_enabled`, `enable_math`, `enable_darkmode`, `al_folio.features.cv.enabled`) **and** (b) the relevant per-page front matter is set (`images:`, `tikzjax`, `chart.*`, `mermaid.*`, `giscus_comments`, `layout: distill|cv`). If a feature silently doesn't appear, check both layers — tags emit empty strings when disabled, not errors.
- **Don't delete v1 config contract keys.** `al_folio.api_version`, `style_engine`, `tailwind.{version,css_entry,preflight}`, `distill.{engine,source}` are required; `upgrade audit` will block on them.
- **Local overrides are tracked.** If you shadow a gem-owned file in your own `_layouts`/`_includes`/`_sass`, run `docker compose exec jekyll bundle exec al-folio upgrade overrides audit` and commit the resulting `.al-folio-overrides.yml` so future gem updates can flag upstream drift.

## Before pushing

```bash
npm run lint:prettier
npm run lint:style-contract
```

CI also runs visual regression (Playwright), integration scripts, and `upgrade audit`.
