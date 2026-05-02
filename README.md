# Christopehr Steinberg — Personal Website

A [Quarto](https://quarto.org) website (R / `knitr` engine) with three pages: home, research, and a CV link.

## Structure

```
.
├── _quarto.yml             # site config, navbar, knitr engine
├── index.qmd               # home / about page
├── research.qmd            # publications, working papers, works in progress
├── styles.css              # custom styling
├── panache.toml            # panache formatter config
├── .pre-commit-config.yaml # R + panache + render hooks
└── assets/
    ├── cv.pdf              # hosted CV (placeholder)
    └── profile.jpg         # profile photo (placeholder)
```

## Requirements

- [R](https://cran.r-project.org/) (`brew install r`)
- [Quarto](https://quarto.org/docs/get-started/) (`brew install quarto`)
- [pre-commit](https://pre-commit.com/) (`brew install pre-commit`)
- [panache](https://github.com/jolars/panache) — Quarto/Markdown formatter (the pre-commit hook installs it via npm automatically; for editor/CLI use also install one of: `cargo install panache`, `uv tool install panache-cli`)

## First-time setup

```r
# from R, in the project root
install.packages(c("renv", "precommit"))
renv::init()
precommit::install_precommit()   # ensures the pre-commit binary is on PATH
```

```bash
pre-commit install               # wire git hooks
pre-commit run --all-files       # smoke-test all hooks
```

The first run downloads and caches the R hook environments (styler, lintr, etc.) — this takes a few minutes once.

## Develop

```bash
quarto preview        # live-reloading dev server
quarto render         # build to _site/
```

## Hooks (configured in `.pre-commit-config.yaml`)

- **lorenzwalthert/precommit** — `style-files` (styler on `.qmd`/`.R`), `parsable-R`, `no-browser-statement`, `no-debug-statement`, `spell-check`
- **jolars/panache** — `panache-format` for Quarto/Markdown formatting
- **pre-commit/pre-commit-hooks** — `check-added-large-files`, `end-of-file-fixer`, `trailing-whitespace`, `check-yaml`, `check-merge-conflict`
- **local** — `forbid-to-commit` (blocks `.Rhistory` etc.) and `quarto-render` (final site render)

## Replace placeholders

- `assets/cv.pdf` — drop in the real CV PDF.
- `assets/profile.jpg` — replace with a real headshot.
- Update social links in `_quarto.yml` and `index.qmd`.

## Deploy (Netlify)

This repo is configured to deploy to Netlify via the [Quarto Netlify Build Plugin](https://github.com/quarto-dev/netlify-plugin-quarto). Netlify renders the site on every push to `main` — no GitHub Action needed.

Files involved:

- `netlify.toml` — sets `publish = "_site"` and registers the plugin.
- `package.json` — declares the `@quarto/netlify-plugin-quarto` dependency.
- `_publish.yml` — records the Netlify site `id` / `url` (filled in after first publish).
- `_freeze/` — committed cache of R computations. Netlify build servers cannot execute R/Python/Julia, so all code must be frozen locally before push.

### One-time setup

1. Render locally so `_freeze/` is populated, then commit it:

   ```bash
   quarto render
   git add _freeze && git commit -m "freeze computations"
   ```

2. In Netlify: **Add new site → Import an existing project**, pick this repo, then on the build settings screen:
   - **Build command:** *(leave blank — the plugin handles it)*
   - **Publish directory:** `_site`

3. Trigger the first deploy. Netlify will install the plugin, run `quarto render`, and publish `_site/`.

4. (Optional) copy the new site's `id` and `url` from the Netlify dashboard into `_publish.yml` so `quarto publish netlify` works locally too.

### Subsequent deploys

```bash
quarto render        # re-runs any changed code, updates _freeze/
git add -A && git commit -m "..." && git push
```

Netlify rebuilds and deploys automatically.
