# taskpapr.github.io

This is the **documentation and marketing website** for [taskpapr](https://taskpapr.com) — a minimal, paper-inspired task board.

The site is built with [Jekyll](https://jekyllrb.com) and the [just-the-docs](https://just-the-docs.com) theme. It is hosted on [GitHub Pages](https://pages.github.com) at **https://taskpapr.github.io**.

> **This is a separate repo from the app.** The app source code lives at [github.com/taskpapr/taskpapr](https://github.com/taskpapr/taskpapr). This repo (`taskpapr.github.io`) is only the website and docs.

---

## Repository structure

```
taskpapr.github.io/
├── _config.yml           # Site config: title, theme, plugins, nav
├── Gemfile               # Ruby dependencies (Jekyll + plugins)
├── index.md              # Homepage
├── docs/
│   ├── getting-started.md   # Getting started guide
│   └── features/
│       └── index.md         # Features overview
├── _layouts/
│   └── default.html         # Base HTML layout (wraps just-the-docs)
├── assets/
│   └── css/
│       └── style.scss        # Paper aesthetic overrides
└── README.md             # This file
```

---

## Running locally

### Prerequisites

You need Ruby ≥ 3.1 and Bundler installed.

```bash
# macOS with Homebrew
brew install ruby
gem install bundler
```

### Setup

```bash
# Clone the repo
git clone https://github.com/taskpapr/taskpapr.github.io.git
cd taskpapr.github.io

# Install dependencies
bundle install
```

### Serve

```bash
bundle exec jekyll serve
```

Then open http://localhost:4000 in your browser. The site reloads automatically when you save files.

### Build (static output)

```bash
bundle exec jekyll build
# Output is in ./_site/
```

---

## How to update the docs

All documentation lives in the `docs/` directory as Markdown files.

### Add a new page

1. Create a new `.md` file in `docs/` (or a subdirectory).
2. Add front matter at the top:

```yaml
---
layout: default
title: My New Page
nav_order: 4        # controls sidebar order
parent: Features    # set if this is a child page
---
```

3. Write your content in Markdown below the front matter.
4. Commit and push — GitHub Actions will build and deploy automatically.

### Edit an existing page

Just edit the Markdown file and push. That's it.

### Add a subdirectory (section)

1. Create a new directory under `docs/`.
2. Add an `index.md` with `has_children: true` in the front matter:

```yaml
---
layout: default
title: My Section
nav_order: 5
has_children: true
---
```

3. Add child pages in the same directory with `parent: My Section` in their front matter.

---

## Deployment

Deployment is fully automatic via GitHub Pages:

- Every push to `main` triggers a GitHub Actions build.
- The built site is deployed to https://taskpapr.github.io within a few minutes.
- Check the **Actions** tab in the repo for build status.

### First-time setup (one-time)

After creating the repo, enable GitHub Pages:

1. Go to **Settings → Pages**
2. Set source to: **Deploy from branch**
3. Branch: `main`, folder: `/ (root)`
4. Click **Save**

---

## Theme & styling

The site uses [just-the-docs](https://just-the-docs.com) via `remote_theme`.

Paper aesthetic overrides are in `assets/css/style.scss`. The colour palette uses warm parchment tones (`#f5f0e8` background, `#3a3228` text) to match the taskpapr app aesthetic.

To make styling changes:
1. Edit `assets/css/style.scss`
2. Run `bundle exec jekyll serve` to preview locally
3. Push when happy

---

## Licence

The taskpapr documentation and website content is © taskpapr contributors.
