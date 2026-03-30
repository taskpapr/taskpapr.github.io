---
layout: default
title: Deployment
nav_order: 5
---

# Deployment
{: .no_toc }

Production self-hosting guide for taskpapr.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

This guide walks through deploying taskpapr in production on a Linux server. The reference architecture uses:

- **EC2** (or any Linux VPS)
- **systemd** (process management + auto-restart)
- **Traefik** (HTTPS reverse proxy)
- **SQLite** (single-file database)

The same approach works on DigitalOcean, Linode, Hetzner, or any provider where you have root access.

---

## Prerequisites

Before starting, you need:

1. **A Linux server** with root/sudo access
2. **A domain name** pointing to your server's IP address
3. **SSH access** configured
4. **Traefik** already running on the server (or install it alongside taskpapr)

For DNS, add an A record:
```
taskpapr.yourdomain.com  →  <your server's public IP>
```

For Traefik, see [Traefik's getting started guide](https://doc.traefik.io/traefik/getting-started/quick-start/) if you don't have it installed yet.

---

## Server setup

SSH into your server and follow these steps.

### Install Node.js 22.5+

```bash
# Install Node.js 22.x via NodeSource
curl -fsSL https://deb.nodesource.com/setup_22.x | sudo -E bash -
sudo apt-get install -y nodejs

# Verify
node --version  # should show v22.5.0 or higher
```

### Create a system user

```bash
sudo useradd -r -m -d /opt/taskpapr -s /bin/bash taskpapr
```

This creates a dedicated user with a home directory at `/opt/taskpapr`.

### Clone the repository

```bash
sudo su - taskpapr
git clone https://github.com/taskpapr/taskpapr.git /opt/taskpapr
cd /opt/taskpapr
npm install --production
exit
```

---

## Environment variables

Create a `.env` file with your production configuration:

```bash
sudo nano /opt/taskpapr/.env
```

Paste (filling in your values):

```dotenv
# Required
SESSION_SECRET=<generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))">
NODE_ENV=production
PORT=3033

# Database
DB_PATH=/opt/taskpapr/data/taskpapr.db

# GitHub OAuth (if using)
GITHUB_CLIENT_ID=your_github_client_id
GITHUB_CLIENT_SECRET=your_github_client_secret
GITHUB_CALLBACK_URL=https://taskpapr.yourdomain.com/auth/github/callback

# OIDC / SSO (if using)
OIDC_ISSUER=https://auth.yourdomain.com/application/o/taskpapr/
OIDC_CLIENT_ID=your_oidc_client_id
OIDC_CLIENT_SECRET=your_oidc_client_secret
OIDC_CALLBACK_URL=https://taskpapr.yourdomain.com/auth/oidc/callback
OIDC_TRUST_IDP=true
```

Set permissions:

```bash
sudo chmod 600 /opt/taskpapr/.env
sudo chown taskpapr:taskpapr /opt/taskpapr/.env
```

See [Authentication](authentication.md) for the full `.env` reference.

---

## systemd service

Create a systemd unit file to manage the taskpapr process:

```bash
sudo nano /etc/systemd/system/taskpapr.service
```

Paste:

```ini
[Unit]
Description=taskpapr — minimal task board
After=network.target

[Service]
Type=simple
User=taskpapr
WorkingDirectory=/opt/taskpapr
ExecStart=/usr/bin/node server.js
Restart=on-failure
RestartSec=10
StandardOutput=journal
StandardError=journal

# Security hardening
NoNewPrivileges=true
PrivateTmp=true
ProtectSystem=strict
ProtectHome=true
ReadWritePaths=/opt/taskpapr/data

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable taskpapr
sudo systemctl start taskpapr
sudo systemctl status taskpapr
```

The service will start automatically on boot and restart if it crashes.

---

## Traefik configuration

taskpapr needs to be exposed via Traefik for HTTPS.

Create a dynamic configuration file for Traefik's file provider:

```bash
sudo nano /etc/traefik/dynamic/taskpapr.yaml
```

Paste:

```yaml
http:
  routers:
    taskpapr:
      rule: "Host(`taskpapr.yourdomain.com`)"
      service: taskpapr
      entryPoints:
        - websecure
      tls:
        certResolver: letsencrypt

  services:
    taskpapr:
      loadBalancer:
        servers:
          - url: "http://localhost:3033"
```

Traefik will detect the file automatically (if file provider watch is enabled). No restart needed.

{: .note }
> If you use a different Traefik setup (e.g., Docker labels), adapt accordingly. The key is to route `taskpapr.yourdomain.com` → `http://localhost:3033`.

---

## Verify deployment

Visit `https://taskpapr.yourdomain.com` in your browser.

- If you configured auth, you should see the login page.
- If you left auth variables empty, you should see the board immediately (single-user mode).

The first user to log in becomes the admin and can manage the whitelist from `/admin`.

---

## Logs

View service logs:

```bash
# Live stream
sudo journalctl -u taskpapr -f

# Last 100 lines
sudo journalctl -u taskpapr -n 100

# Logs from the last hour
sudo journalctl -u taskpapr --since "1 hour ago"
```

---

## Database backup

The database is a single SQLite file at `/opt/taskpapr/data/taskpapr.db`.

Back it up by copying it:

```bash
sudo cp /opt/taskpapr/data/taskpapr.db /opt/taskpapr/data/taskpapr.db.backup
```

For automated backups, add a cron job:

```bash
sudo crontab -e
```

Add:

```
# Daily backup at 3am
0 3 * * * cp /opt/taskpapr/data/taskpapr.db /opt/taskpapr/data/taskpapr-$(date +\%Y\%m\%d).db
```

{: .warning }
> SQLite databases can be copied while the app is running, but for critical production use, consider stopping the service first or using `sqlite3 .backup`.

---

## Upgrading

When a new version is released:

```bash
sudo su - taskpapr
cd /opt/taskpapr
git pull
npm ci --production
exit

sudo systemctl restart taskpapr
```

Check the logs to confirm the new version started successfully:

```bash
sudo journalctl -u taskpapr -n 50
```

---

## Docker in production

If you prefer Docker, use the official image:

```bash
docker run -d \
  --name taskpapr \
  --restart unless-stopped \
  -p 3033:3033 \
  -v taskpapr-data:/data \
  -e DB_PATH=/data/taskpapr.db \
  -e SESSION_SECRET=<generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"> \
  -e NODE_ENV=production \
  ghcr.io/taskpapr/taskpapr:latest
```

Or use `docker-compose.yml`:

```yaml
version: '3.8'

services:
  taskpapr:
    image: ghcr.io/taskpapr/taskpapr:latest
    container_name: taskpapr
    restart: unless-stopped
    ports:
      - "3033:3033"
    volumes:
      - taskpapr-data:/data
    environment:
      DB_PATH: /data/taskpapr.db
      SESSION_SECRET: <generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))">
      NODE_ENV: production
      # Add auth variables as needed (see Authentication docs)

volumes:
  taskpapr-data:
```

Start:

```bash
docker compose up -d
```

Update:

```bash
docker compose pull
docker compose up -d
```

Data persists in the `taskpapr-data` volume. To back up:

```bash
docker run --rm -v taskpapr-data:/data -v $(pwd):/backup alpine tar czf /backup/taskpapr-backup.tar.gz -C /data .
```

---

## Troubleshooting

### Service won't start

Check logs:

```bash
sudo journalctl -u taskpapr -n 50
```

Common issues:
- Missing `.env` file or incorrect permissions
- Port 3033 already in use (check with `sudo lsof -i :3033`)
- Node.js version < 22.5.0

### Traefik can't reach taskpapr

Verify taskpapr is listening:

```bash
curl http://localhost:3033
```

If that works but Traefik can't reach it, check:
- Traefik dynamic config file syntax
- Traefik logs: `sudo docker logs traefik` (if running Traefik in Docker)

### "No route to host" errors

Check firewall rules:

```bash
sudo ufw status
```

Ensure port 80 and 443 are open for Traefik. Port 3033 should NOT be open to the internet — only Traefik needs to reach it internally.

---

## Security notes

- **Always use HTTPS** in production (Traefik + Let's Encrypt handles this)
- **Keep `SESSION_SECRET` secret** — do not commit it to git
- **Use `OIDC_TRUST_IDP=true`** when using your own IdP (e.g. Authentik) to skip redundant whitelist checks
- **Run taskpapr as a non-root user** (the systemd unit file does this)
- **Restrict file permissions** on `.env` (`chmod 600`)

---

## Next steps

- Set up [Telegram notifications](../features/telegram.md)
- Explore the [API](../features/api-and-webhooks.md) for integrations
- Configure the [MCP server](../features/mcp.md) for AI assistant access