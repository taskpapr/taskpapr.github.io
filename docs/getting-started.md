---
layout: default
title: Getting Started
nav_order: 2
---

# Getting Started
{: .no_toc }

Everything you need to go from zero to your first task board.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Requirements

- **Node.js 22.5 or later** — taskpapr uses the built-in `node:sqlite` module introduced in Node 22.5. Because it ships inside Node itself, there is no native compilation step and no dependency on system libraries. If `node --version` shows anything below `v22.5.0`, update Node first.
- **npm** — comes with Node, no separate install needed.
- **macOS or Linux** — taskpapr runs anywhere Node runs. The launchd auto-start section below is macOS-only; Linux users can use systemd (see [Deployment](deployment.md)).

Check your Node version:

```bash
node --version
```

---

## Quick start — local Mac setup

```bash
git clone https://github.com/taskpapr/taskpapr.git
cd taskpapr
npm install
npm start
```

Then open **[http://localhost:3033](http://localhost:3033)** in your browser.

That's it. No database to configure, no environment variables required. taskpapr starts in single-user mode by default — no login page, straight to the board.

{: .note }
> The database is created automatically at `data/papr.db` on first run. It is a single SQLite file. Back it up by copying it.

---

## Alfred shortcut

The fastest way to open taskpapr is a one-keystroke Alfred workflow. Once set up, pressing your Alfred hotkey and typing `td` opens the board instantly.

1. Open **Alfred Preferences** → **Features** → **Web Search**
2. Click **Add Custom Search**
3. Fill in:

   | Field | Value |
   |---|---|
   | Search URL | `http://localhost:3033` |
   | Title | `taskpapr` |
   | Keyword | `td` |

4. Click **Save**

Now press your Alfred hotkey, type `td`, press `Enter` — the board opens.

Alternatively, create a **Simple URL** workflow in Alfred's Workflows tab with the same keyword and URL if you prefer a workflow over a web search entry.

---

## Auto-start on login (macOS)

taskpapr ships with a launchd plist that starts the server automatically when you log in and restarts it if it crashes.

**Install:**

```bash
cp com.papr.server.plist ~/Library/LaunchAgents/
launchctl load ~/Library/LaunchAgents/com.papr.server.plist
```

The server is now running and will start automatically on every login.

**Stop it:**

```bash
launchctl unload ~/Library/LaunchAgents/com.papr.server.plist
```

**Check logs:**

```bash
tail -f ~/Library/Logs/papr.log        # stdout
tail -f ~/Library/Logs/papr.error.log  # stderr
```

{: .important }
> The plist file contains hardcoded paths. Open it in a text editor and update the `ProgramArguments` and `WorkingDirectory` keys to match where you cloned the repo, and the path to your Node installation (`which node` will tell you).

---

## Docker

If you would rather not install Node directly, Docker is a straightforward alternative.

**Single container:**

```bash
docker run -d \
  --name taskpapr \
  --restart unless-stopped \
  -p 3033:3033 \
  -v taskpapr-data:/data \
  -e DB_PATH=/data/papr.db \
  ghcr.io/taskpapr/taskpapr:latest
```

Open **[http://localhost:3033](http://localhost:3033)**.

**docker-compose:**

Copy the example file and start the stack:

```bash
cp docker-compose.yml.example docker-compose.yml
docker compose up -d
```

The compose file maps port `3033` and persists data in a named volume (`taskpapr-data`). To add auth environment variables, add an `env_file` or `environment` block to the service — see [Authentication](authentication.md) for the full variable reference.

**Stop:**

```bash
docker compose down
```

Data is preserved in the volume. To also remove the data:

```bash
docker compose down -v
```

---

## Self-hosting on a server

For a production setup on a Linux server — with HTTPS, automatic restarts, and a reverse proxy — see the full walkthrough in [Deployment](deployment.md).

The reference architecture uses **EC2 + systemd + Traefik**, but the same approach works on any Linux VPS.

---

## Single-user vs multi-user mode

### Single-user mode (default)

If you set no auth environment variables, taskpapr starts in single-user mode:

- No login page
- No session
- Opens straight to the board
- No configuration needed

This is the right choice for personal use on your own machine.

### Multi-user mode

Set one or both of the following to enable login:

| What you want | Variables to set |
|---|---|
| Sign in with GitHub | `GITHUB_CLIENT_ID` + `GITHUB_CLIENT_SECRET` |
| Sign in with SSO (Authentik or any OIDC provider) | `OIDC_ISSUER` + `OIDC_CLIENT_ID` + `OIDC_CLIENT_SECRET` |

When any auth variable is present, taskpapr shows a login page. Users must authenticate before accessing the board.

You can run both providers simultaneously — both buttons will appear on the login page.

See [Authentication](authentication.md) for step-by-step setup instructions and the full environment variable reference.
