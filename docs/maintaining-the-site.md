---
layout: default
title: Maintaining the Site
nav_order: 8
---

# Maintaining the Site
{: .no_toc }

A personal reference guide for keeping the docs in sync with the app.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## How the site works

The website lives in a separate repository from the app:

| Repo | What it is |
|---|---|
| `~/Development/taskpapr.github.io` | This site — docs and marketing |
| `~/Development/taskpapr` | The app itself |

The site is built with **Jekyll** — a tool that converts Markdown files (`.md`) into HTML. You write plain text; Jekyll handles the rest.

**Publishing is automatic.** Every time you push a commit to the `main` branch, GitHub Pages rebuilds the site and deploys it. The whole process takes about 1–2 minutes. You never need to run any build commands unless you want to preview changes locally first.

The theme is [just-the-docs](https://just-the-docs.com/), which handles the sidebar navigation, search, and overall layout. Navigation is controlled entirely by front matter in each `.md` file — no config file to update when you add a page.

---

## Running the site locally (optional)

You don't need to run the site locally — you can just push and check the live site. But local preview is useful when making layout changes or adding multiple pages at once.

### First-time setup

```bash
# Install Ruby (if not already installed)
brew install ruby

# Make sure the brew Ruby is on your PATH
echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# Install Bundler
gem install bundler

# Install the site's dependencies (run once, from the repo root)
cd ~/Development/taskpapr.github.io
bundle install
```

### Start the local server

```bash
cd ~/Development/taskpapr.github.io
bundle exec jekyll serve
```

Open **http://localhost:4000** in your browser.

Changes to `.md` files are picked up automatically — just save the file and refresh the browser. You don't need to restart the server.

Stop it with `Ctrl+C`.

---

## Documentation images and screenshots

Visuals live under **`assets/images/docs/`** — real **PNG** captures from a running instance (`board-overview.png`, `today-tile.png`, `recurring-urgency.png`, `dormant-ghost-pill.png`, `goals-smart-tile.png`).

### When to refresh

- The board, tiles, Today view, goals chrome, or recurring-task visuals change in a way readers would notice.
- You ship a release that intentionally updates colours, spacing, or typography.

Add a release-checklist tick (see below) so it is not forgotten.

### Capturing real screenshots

1. Run taskpapr locally against a **throwaway DB** (`PORT=3099 DB_PATH=/tmp/taskpapr-screenshots.db node server.js` in the app repo) — never point a capture session at a real instance.
2. Load **stable demo data** so crops stay comparable between releases:
   ```bash
   node demo-data/generate.js | curl -X POST 'http://localhost:3099/api/import?mode=replace' \
     -H "Content-Type: application/json" --data-binary @-
   ```
   (from the app repo). This is the exact board every screenshot on this site was captured from — Work/Personal/Project/Errands tiles plus a "Recurring examples" tile whose four tasks are tuned to the calm/amber/orange/red steps of the spinning-plates urgency gradient. See `demo-data/README.md` in the app repo for what's in it. Timestamps are computed relative to "now" on every run, so the urgency gradient always looks right — no manual date-patching needed (that used to be necessary here; `/api/import` now round-trips `created_at` correctly).
3. Use a **fixed viewport width** (around 1200–1400 CSS px) and the same zoom level (100%) each time.
4. Crop tightly to the UI element the doc page is illustrating — tile, panel, or header — rather than shipping full-page screenshots with the app chrome around them.
5. Save as **PNG** into `assets/images/docs/` with a descriptive name (`board-overview.png`, `today-tile.png`, …). Keep the long edge around 1200px or less so the repo and mobile readers stay fast; WebP is fine too if you have `cwebp`/ImageMagick available for the ~80%-quality re-encode.
6. In the Markdown page, use a `<figure class="doc-figure">` (or the same pattern as existing feature pages) with **`{{ '/assets/images/docs/your-file.png' | relative_url }}`** so `baseurl` stays correct on GitHub Pages, and set `width`/`height` to the actual captured pixel dimensions.
7. Write **meaningful `alt` text** (what the reader should learn from the image, not just "screenshot").
8. Tear down the throwaway server and delete the temp DB when done.

### Diagrams and UI vignettes

Some pages use a small hand-drawn **SVG** instead of a screenshot where a stylised illustration communicates better than pixel-perfect UI (or where the real UI is awkward to capture in isolation). **UI vignettes** are small HTML blocks in **`_includes/`** (e.g. `doc_vignette_wip.html`) styled in **`assets/css/style.scss`** (`.doc-vignette`, `.doc-vignette-task`, …). Update SCSS if the app's palette or chrome changes materially.

### Includes in Markdown

Pages can use:

```liquid
{% include doc_vignette_wip.html %}
```

Jekyll renders Liquid before Markdown conversion. If an include ever appears as raw text in the built HTML, confirm you are building with Jekyll from this repo’s `Gemfile` / GitHub Actions (not a stripped-down Markdown previewer that skips Liquid).

---

## Release checklist

Run through this each time a new version of taskpapr ships.

```
[ ] Update the version number wherever it appears
    - Search for the old version: grep -r "v0.45.8" .
    - Update index.md (homepage hero/footer) and anywhere else it appears

[ ] New feature added?
    - Create or update the relevant docs/features/ page
    - Make sure docs/features/index.md table still accurately describes all pages

[ ] Existing feature changed behaviour?
    - Update the affected docs page

[ ] New environment variable added?
    - Update the env var table in docs/authentication.md

[ ] New API endpoint added?
    - Update docs/features/api-and-webhooks.md

[ ] Getting-started flow changed?
    - Update docs/getting-started.md

[ ] Homepage feature highlights still accurate?
    - Open index.md, check the features grid and "why taskpapr" section

[ ] Board / task UI visuals changed for readers?
    - Refresh images in assets/images/docs/ and/or vignette HTML–CSS (see “Documentation images…” above)

[ ] Commit and push:
    git add .
    git commit -m "docs: update for vX.Y.Z"
    git push

[ ] Wait ~2 minutes, then check https://docs.taskpapr.com

[ ] If something looks wrong, check the Actions tab in the GitHub repo for build errors
```

---

## How to add a new documentation page

1. **Create the file** in the right folder:
   - Top-level doc (e.g. a new guide): `docs/my-new-page.md`
   - Feature sub-page: `docs/features/my-new-feature.md`

2. **Add front matter** at the top of the file (see template below)

3. **Write the content** in Markdown

4. **Push** — the page appears in the sidebar automatically

That's it. No config file to update, no menus to register. The sidebar is built from the front matter.

### Front matter template

Copy and paste this at the top of any new page, filling in the values:

**Top-level page** (e.g. `docs/my-guide.md`):
```yaml
---
layout: default
title: My Guide
nav_order: 7
---
```

**Feature sub-page** (e.g. `docs/features/my-feature.md`):
```yaml
---
layout: default
title: My Feature
parent: Features
nav_order: 8
---
```

**Front matter fields:**

| Field | What it does |
|---|---|
| `layout: default` | Always use `default` for docs pages |
| `title` | The page title — appears in the sidebar and browser tab |
| `nav_order` | Controls the order in the sidebar (lower = higher up) |
| `parent` | Name of the parent page — must match that page's `title` exactly |
| `nav_exclude: true` | Hides the page from the sidebar (used by the homepage) |
| `has_children: true` | Marks a page as a section with sub-pages (used by Features index) |

### Adding the table of contents

To add a TOC to any page, paste this block immediately after the page heading:

```markdown
## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}
```

---

## How to edit existing content

1. Open the relevant `.md` file in the `~/Development/taskpapr.github.io` folder
2. Edit the content
3. Save
4. Push:
   ```bash
   git add .
   git commit -m "docs: fix typo in authentication page"
   git push
   ```

No build step. No preview required for small edits. The live site updates in ~2 minutes.

---

## Troubleshooting

**Build failed — site not updating**
Check the **Actions** tab in the GitHub repo (`github.com/taskpapr/taskpapr.github.io/actions`). The error will be shown there. Common causes:
- Malformed front matter (missing `---` delimiters, bad indentation)
- A Liquid template error (usually a missing `%}` or `}}`)
- A broken `include` or `layout` reference

**Page not appearing in the sidebar**
- Check `nav_order` is set and is a number
- Check `parent` exactly matches the parent page's `title` (case-sensitive)
- Check there's no `nav_exclude: true` on the page

**Changes not showing on the live site**
- Wait 2 minutes and hard-refresh: `Cmd+Shift+R`
- Check the Actions tab to confirm the build succeeded
- Make sure you pushed to `main` (not another branch)

**Local server won't start**
```bash
bundle install   # re-run if Gemfile.lock is out of date
bundle exec jekyll serve
```
If you get Ruby version errors, check `brew install ruby` succeeded and the brew Ruby is on your PATH (see first-time setup above).

**Sidebar order is wrong**
`nav_order` values don't need to be sequential — they just need to be in the right relative order. You can use `1`, `2`, `3` or `10`, `20`, `30` — whatever is easiest to maintain. Lower numbers appear higher in the sidebar.

---

## Custom domain: docs.taskpapr.com

This site is served at **docs.taskpapr.com**, separate from the app (`app.taskpapr.com`, `staging.taskpapr.com`) and the marketing apex. DNS for `taskpapr.com` is in Cloudflare, same as the app subdomains (see `taskpapr-infra/docs/runbooks/`).

Setup, for reference (already done — see below if it ever needs redoing, e.g. after a domain transfer):

1. **Repo:** a `CNAME` file at the repo root containing just:
   ```
   docs.taskpapr.com
   ```
   `_config.yml`'s `url:` must match (`https://docs.taskpapr.com`) so canonical URLs, the sitemap, and SEO tags are correct.

2. **Cloudflare:** add a DNS record on the `taskpapr.com` zone:
   ```
   Type:   CNAME
   Name:   docs
   Target: taskpapr.github.io
   Proxy:  DNS only (grey cloud) until GitHub issues the cert — see step 4
   TTL:    Auto
   ```
   This is a subdomain, so it's a plain CNAME — no ALIAS/ANAME or apex trickery needed (that's only required for `@`/root domains, which point at GitHub's A/AAAA records instead).

3. **GitHub:** repo Settings → Pages → Custom domain → enter `docs.taskpapr.com` → Save.

   {: .warning }
   > **This step does not happen automatically just by pushing the `CNAME` file** — that only applies to the legacy "Deploy from a branch" Pages builder. This site's `build_type` is `workflow` (`.github/workflows/pages.yml`, using `actions/deploy-pages`), and for workflow-based deploys GitHub does not scan the published output for a `CNAME` file to infer the custom domain. Skipping this step is a silent failure, not an error: DNS resolves, TLS terminates, `Server: GitHub.com` comes back — and GitHub Pages 404s anyway, because the domain was never associated with this repo. Confirm it's set with:
   > ```bash
   > gh api repos/taskpapr/taskpapr.github.io/pages --jq '.cname'
   > # should print docs.taskpapr.com, not null
   > ```
   > If it prints `null`, set it directly: `gh api -X PUT repos/taskpapr/taskpapr.github.io/pages -f cname='docs.taskpapr.com'`.

4. **Wait for DNS to propagate**, then check the **Enforce HTTPS** checkbox in the same Settings → Pages screen once it's available (GitHub greys it out until the domain resolves and it has issued a Let's Encrypt cert — can take a few minutes to an hour). Or via API once `https_certificate.state` is `approved`: `gh api -X PUT repos/taskpapr/taskpapr.github.io/pages -F https_enforced=true`.

5. **Cloudflare proxy (optional):** once HTTPS is enforced and working, the record can be switched to proxied (orange cloud) if you want Cloudflare's CDN/caching in front of it. If you do, set Cloudflare SSL/TLS mode to **Full** (not Flexible) or you'll get a redirect loop.

`https://taskpapr.github.io` keeps working alongside the custom domain — GitHub doesn't disable the default `github.io` URL when a custom domain is set.
