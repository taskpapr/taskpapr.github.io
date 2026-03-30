# CONTEXT

## What this repo is
Marketing and docs website for taskpapr. Jekyll + just-the-docs theme. Hosted on GitHub Pages at https://taskpapr.github.io. Separate from the app repo (github.com/taskpapr/taskpapr).

## Stack
- Jekyll static site generator
- remote_theme: just-the-docs/just-the-docs@v0.10.0
- Hosted on GitHub Pages (auto-deploy on push to main)
- Ruby / Bundler for local dev

## Run locally
```
bundle install
bundle exec jekyll serve
# opens at http://localhost:4000
```

## Repo structure
```
_config.yml                       Site config, theme, plugins, nav
Gemfile                           Ruby deps
index.md                          Homepage (layout: home, nav_exclude: true)
_layouts/home.html                Wraps just-the-docs default; disables nav
_includes/head_custom.html        Injects fonts (Source Serif 4, Lato) and style.css
assets/css/style.scss             All custom CSS (paper aesthetic overrides)
docs/getting-started.md           Getting Started page (nav_order: 2)
docs/authentication.md            Authentication page (nav_order: 4)
docs/deployment.md                Deployment page (nav_order: 5)
docs/features/index.md            Features overview (nav_order: 3, has_children: true)
```

## Brand / design language
Tagline: "A minimal, paper-inspired task board. No noise, no friction."
Aesthetic: warm parchment tones
  paper-bg: #f5f0e8
  paper-text: #3a3228
  paper-accent: #7c6f4a (earthy gold)
Fonts: Source Serif 4 (headings, serif), Lato 300/400/700 (body, sans)
Homepage CSS classes: hp-hero, hp-btn, hp-why-grid, hp-features-grid, hp-feature-card, hp-section, hp-selfhost, hp-callout, hp-footer

## The app (taskpapr)
- Self-hosted Node.js task board
- Port 3033 by default
- SQLite via node:sqlite (Node 22.5+ required, no native compilation)
- Single-user by default (no login); multi-user via GitHub OAuth or OIDC
- Docker image: ghcr.io/taskpapr/taskpapr:latest
- Current version shown on homepage: v0.33.0

## Key features of the app
1. **Canvas** — infinite canvas, freely draggable tiles
2. **Done** — completed tasks stay visible, faded
3. **WIP** — amber left stripe for in-progress tasks
4. **Plates** — spinning plates; recurring tasks with visual urgency heat (amber → orange → red)
5. **Rot** — tasks fade over time if untouched (visual only, no alerts)
6. **Goals** — outcomes view above the noise; smart-tiles on canvas showing task progress
7. **Telegram** — daily digest + quick-capture from chat
8. **API & Webhooks** — REST API with Bearer token auth; webhook actions for n8n/Zapier/Make
9. **MCP Server** — 9 tools for Claude Desktop, Cline, or any MCP-compatible AI assistant
10. **Import/Export** — portable JSON backup/restore with merge or replace mode

## Docs pages created
| File | Title | nav_order | Status |
|---|---|---|---|
| docs/getting-started.md | Getting Started | 2 | ✅ Created |
| docs/features/index.md | Features (overview) | 3 (parent) | ✅ Created |
| docs/features/the-board.md | The Board | 3.1 | ✅ Created |
| docs/features/recurring-tasks.md | Recurring Tasks | 3.2 | ✅ Created |
| docs/features/goals.md | Goals | 3.3 | ✅ Created |
| docs/features/telegram.md | Telegram | 3.4 | ✅ Created |
| docs/features/api-and-webhooks.md | API & Webhooks | 3.5 | ✅ Created |
| docs/features/mcp.md | MCP Server | 3.6 | ✅ Created |
| docs/features/import-export.md | Import & Export | 3.7 | ✅ Created |
| docs/authentication.md | Authentication | 4 | ✅ Created |
| docs/deployment.md | Deployment | 5 | ✅ Created |
| docs/maintaining-the-site.md | Maintaining the Site | 6 | ✅ Created |

## Docs pages still to create

None — all pages are complete.

## Navigation config (_config.yml)
- Aux link top-right: "Launch App" → https://taskpapr.com (new tab)
- Nav external link: GitHub → https://github.com/taskpapr
- Search enabled
- color_scheme: taskpapr (commented out — needs assets/css/color_schemes/taskpapr.scss first)

## Authentication detail (for writing auth-related docs)
- **Single-user mode**: no auth env vars set → no login, opens straight to board
- **Multi-user mode**: set GITHUB_CLIENT_ID+SECRET and/or OIDC_ISSUER+CLIENT_ID+SECRET
- **Whitelist**: by default users must be on the whitelist; managed at /admin
- **OIDC_TRUST_IDP=true**: skips whitelist check for OIDC logins (recommended with Authentik)
- **First user bootstrap**: first login (any provider) auto-creates admin, skips whitelist
- **GitHub email scope**: app requests user:email scope; users without an accessible email are rejected
- Supports both providers simultaneously (both buttons appear on login page)

## Deployment detail (for writing deployment docs)
- Reference architecture: EC2 (or any VPS) + systemd + Traefik + SQLite
- Default port: 3033
- System user: taskpapr, home at /opt/taskpapr
- DB path default: ./data/papr.db (override with DB_PATH env var)
- Logs via journalctl (-u taskpapr)
- Docker also supported: ghcr.io/taskpapr/taskpapr:latest

## Plugins
jekyll-feed, jekyll-remote-theme, jekyll-seo-tag

## Notes / outstanding TODOs
- color_scheme line in _config.yml is commented out; uncomment once assets/css/color_schemes/taskpapr.scss is created
- Homepage footer shows v0.33.0 and "License TBD" — update when license is decided
- The launchd plist (com.papr.server.plist) is referenced in getting-started as shipping with the app
- Repo structure section above only lists top-level docs; feature sub-pages are all now present under docs/features/
