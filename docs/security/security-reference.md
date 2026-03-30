---
layout: default
title: Security Reference
parent: Security
nav_order: 1
---

# Security Reference
{: .no_toc }

What's built in, what v0.35.0 adds, and how to harden a public-facing deployment.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Part 1 — What's already built in

These protections are in the application code and require no configuration.

### SQL injection

Every database query uses `node:sqlite` prepared statements with parameterised values. There is no SQL string concatenation anywhere in the codebase. The attack surface for SQL injection is zero.

### Authentication and authorisation

- `requireAuth` middleware is applied globally to all routes. Only genuinely public endpoints are exempted (`/login`, `/auth/*`, `/api/telegram/webhook`).
- All data queries are scoped to the authenticated user's ID — a user cannot read or modify another user's data even if they know the ID of another resource.
- Session tokens are opaque random values managed by `express-session`. They cannot be forged.
- Sessions are stored in SQLite, not in-memory. They survive server restarts.

### API key security

- Raw API keys are **never stored**. Only a SHA-256 hash is stored in the database.
- Keys are generated as `tp_<64 hex chars>` (256 bits of entropy), shown to the user exactly once, and cannot be retrieved again.
- The webhook endpoint (`POST /api/webhook`) requires API key authentication. Session cookies are explicitly not accepted, even in single-user mode — this prevents CSRF-style attacks.

### Telegram webhook

- The `/api/telegram/webhook` endpoint is intentionally public (Telegram servers POST to it).
- Set `TELEGRAM_WEBHOOK_SECRET` in `.env` to enable `X-Telegram-Bot-Api-Secret-Token` header validation — only Telegram's servers can deliver updates.
- Messages from unlinked chat IDs are silently ignored. No reply is sent, no enumeration is possible.

### No secrets in source

- All secrets (`SESSION_SECRET`, OAuth credentials, API keys, Telegram tokens) are in `.env`, which is in `.gitignore`.
- `SESSION_SECRET` must be set explicitly. There is no insecure fallback default.

### Minimal dependency surface

The application has a deliberately small dependency list: `express`, `express-session`, `passport` + two strategies, `dotenv`, and the MCP SDK. No ORM, no JWT library, no template engine. Fewer dependencies means fewer CVEs to track.

---

## Part 2 — v0.35.0 application hardening

These features were added in v0.35.0 and are active by default.

### Rate limiting

Three tiers of rate limiting, applied at the application layer:

| Tier | Default | Env var | Applied to |
|---|---|---|---|
| Global | 300 req/min/IP | `RATE_LIMIT_GLOBAL` | All routes |
| Writes | 30 req/min/IP | `RATE_LIMIT_WRITES` | Task/tile/goal/bookmark/webhook creation |
| Auth | 20 req/min/IP | `RATE_LIMIT_AUTH` | `/auth/github`, `/auth/oidc` |

Returns HTTP 429 with a `Retry-After` header when a limit is breached. Self-hosters can raise or lower these values freely via env vars.

### Security headers

`helmet` middleware sets all recommended HTTP security headers on every response:

- `X-Content-Type-Options: nosniff` — prevents MIME sniffing
- `X-Frame-Options: DENY` — prevents clickjacking
- `Referrer-Policy: no-referrer` — no referrer leakage on outbound links
- `X-DNS-Prefetch-Control: off`
- `Permissions-Policy` — disables unused browser features

{: .note }
> Content Security Policy (CSP) is explicitly disabled (`helmet({ contentSecurityPolicy: false })`). The default Helmet CSP blocks inline scripts, which would break all page JavaScript. All other Helmet protections are active.

### Input size validation

Server-side length limits on all write endpoints. Frontend `maxlength` attributes are UX only — these are the enforced server-side limits:

| Field | Default | Env var |
|---|---|---|
| Task/goal/tile/bookmark name | 100 chars | `LIMIT_NAME_LEN` |
| Task title | 500 chars | `LIMIT_TITLE_LEN` |
| Task notes | 50,000 chars (~50KB) | `LIMIT_NOTES_LEN` |

Returns HTTP 400 with a descriptive error message. The JSON body parser is also capped at 200KB.

### Per-user resource quotas

A count check runs before every insert. Returns HTTP 429 if the user has reached their limit:

| Resource | Default | Env var |
|---|---|---|
| Tasks per user | 2,000 | `LIMIT_TASKS` |
| Tiles per user | 50 | `LIMIT_TILES` |
| Goals per user | 50 | `LIMIT_GOALS` |
| Bookmarks per user | 20 | `LIMIT_BOOKMARKS` |

Quotas apply to all insert paths: REST API, webhook, and Telegram quick-capture. Self-hosters can raise all limits freely.

### CI: dependency audit

`npm audit --audit-level=high` runs in CI on every push. High or critical CVEs fail the build. Low and moderate are reported but non-blocking.

---

## Part 3 — Recommended infrastructure pattern

This section is for **public-facing deployments** (exposed to the internet). A private deployment on a home network or LAN doesn't need this — the application defaults are sufficient.

### Overview

```
Internet
    ↓
Cloudflare (free tier)
  Absorbs DDoS, hides your VPS IP, edge rate limiting
    ↓  (only Cloudflare IPs reach your VPS)
VPS firewall
  Port 80/443 open; port 3033 bound to 127.0.0.1 only
    ↓
Traefik
  TLS termination, rate limiting (second layer), HSTS, body size limit
    ↓
taskpapr (port 3033, localhost only)
  Rate limiting (third layer), helmet, input validation, quotas
```

Three independent layers means an attacker must defeat all three. Any single layer is sufficient to protect the layers behind it.

### Why Cloudflare (free tier is enough)

- **Volumetric DDoS** — Cloudflare absorbs bandwidth attacks before they reach your VPS. Your VPS never sees the traffic, so there are no egress charges.
- **Your VPS IP is never exposed** — traffic arrives only through Cloudflare's anycast network. Attackers cannot directly target your VPS IP because they don't know it.
- **Free tier includes**: DDoS protection, CDN, basic rate limiting rules, bot score filtering, and "I'm Under Attack" mode (JS challenge for all visitors, toggleable instantly from the dashboard).

**Setup:**
1. Add your domain to Cloudflare (or transfer it)
2. Set the DNS A record to your VPS IP with the proxy (orange cloud ☁) enabled
3. SSL/TLS → set mode to **Full (strict)** — Cloudflare encrypts to origin, Let's Encrypt provides the cert
4. Enable **Always Use HTTPS**
5. Optionally add a rate limit rule: > 100 req/min from one IP → block for 1 minute

### Traefik configuration

Extend your Traefik dynamic config to add rate limiting, HSTS headers, and a Cloudflare IP allowlist:

```yaml
# traefik-dynamic.yaml
http:
  middlewares:
    taskpapr-ratelimit:
      rateLimit:
        average: 100    # requests per period
        period: 1m
        burst: 50

    taskpapr-secure-headers:
      headers:
        stsSeconds: 31536000
        stsIncludeSubdomains: true
        forceSTSHeader: true
        contentTypeNosniff: true
        frameDeny: true
        referrerPolicy: "no-referrer"

    cloudflare-only:
      ipWhiteList:
        sourceRange:
          # Cloudflare IPv4 (https://www.cloudflare.com/ips/)
          - "173.245.48.0/20"
          - "103.21.244.0/22"
          - "103.22.200.0/22"
          - "103.31.4.0/22"
          - "141.101.64.0/18"
          - "108.162.192.0/18"
          - "190.93.240.0/20"
          - "188.114.96.0/20"
          - "197.234.240.0/22"
          - "198.41.128.0/17"
          - "162.158.0.0/15"
          - "104.16.0.0/13"
          - "104.24.0.0/14"
          - "172.64.0.0/13"
          - "131.0.72.0/22"
          # Cloudflare IPv6
          - "2400:cb00::/32"
          - "2606:4700::/32"
          - "2803:f800::/32"
          - "2405:b500::/32"
          - "2405:8100::/32"
          - "2a06:98c0::/29"
          - "2c0f:f248::/32"

  routers:
    taskpapr:
      rule: "Host(`yourdomain.com`)"
      entryPoints:
        - websecure
      middlewares:
        - cloudflare-only
        - taskpapr-ratelimit
        - taskpapr-secure-headers
      service: taskpapr
      tls:
        certResolver: letsencrypt

  services:
    taskpapr:
      loadBalancer:
        servers:
          - url: "http://127.0.0.1:3033"
```

{: .note }
> Cloudflare's IP ranges change occasionally. Check [cloudflare.com/ips](https://www.cloudflare.com/ips/) when updating your Traefik config.

### VPS firewall

Port 3033 must never be directly reachable from the internet. taskpapr binds to `127.0.0.1:3033` by default — this firewall rule is a belt-and-suspenders check:

```bash
ufw allow 22/tcp    # SSH
ufw allow 80/tcp    # Traefik HTTP
ufw allow 443/tcp   # Traefik HTTPS
ufw deny 3033       # Block direct app access
ufw enable
```

Verify taskpapr is only listening on localhost:

```bash
ss -tlnp | grep 3033
# Should show: 127.0.0.1:3033
# NOT: 0.0.0.0:3033
```

If it shows `0.0.0.0:3033`, set `HOST=127.0.0.1` in your `.env` file.

---

## Part 4 — What stays out of scope (intentionally)

These are security concerns that belong at the infrastructure layer, not in the Node.js application:

| Concern | Where it belongs | Why not in the app |
|---|---|---|
| Volumetric DDoS | Cloudflare | Node.js cannot absorb bandwidth attacks; the process dies before it can reject traffic |
| IP blocking / allowlisting | Cloudflare or Traefik | App-level IP blocking is fragile; the proxy has better visibility |
| Custom WAF rules | Cloudflare | Cloudflare's managed rules cover OWASP Top 10 on the free tier |
| CAPTCHA | OAuth providers | GitHub and OIDC providers already use CAPTCHA on their login flows |
| Bot detection | Cloudflare bot score | Free tier provides basic bot scoring, tunable without app changes |
| TLS / HTTPS | Traefik + Let's Encrypt | Never terminate TLS in the Node.js process directly |

Adding these to the application would increase complexity, increase attack surface, and provide weaker protection than the proxy layer.

---

## Part 5 — Ongoing hygiene

- **`npm audit` in CI** — runs on every push; high/critical CVEs fail the build. Review the output.
- **Dependabot** — enable in your fork: Settings → Code security → Dependabot alerts + Dependabot security updates. Sends PRs for dependency vulnerabilities automatically.
- **Session secret rotation** — if `SESSION_SECRET` is ever compromised, change it and restart. All active sessions will be invalidated; users will need to log in again.
- **Review the user list** — `/admin` shows all registered users. Periodically check for unexpected accounts.
- **Cloudflare IP ranges** — Cloudflare publishes current ranges at [cloudflare.com/ips](https://www.cloudflare.com/ips/). Review the Traefik allowlist when updating your Traefik config.
