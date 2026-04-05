---
layout: default
title: Authentication
nav_order: 6
---

# Authentication
{: .no_toc }

How to configure login for taskpapr.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Single-user mode (default)

If no auth environment variables are set, taskpapr starts in **single-user mode**:

- No login page
- No session management
- Opens straight to the board
- No configuration needed

This is the right choice for personal use on your own machine. There is nothing to configure.

---

## Multi-user mode

Set one or both of the following to enable login:

- `GITHUB_CLIENT_ID` + `GITHUB_CLIENT_SECRET` → "Sign in with GitHub"
- `OIDC_ISSUER` + `OIDC_CLIENT_ID` + `OIDC_CLIENT_SECRET` → "Sign in with SSO"

When any auth variable is present, taskpapr shows a login page. Users must authenticate before accessing the board.

You can run both providers simultaneously — both buttons will appear on the login page.

---

## GitHub OAuth setup

### Create the OAuth App on GitHub

1. Go to **https://github.com/settings/developers**
2. Click **"New OAuth App"** (or open an existing one to edit)
3. Fill in:

   | Field | Value |
   |---|---|
   | Application name | `taskpapr` (or anything you like) |
   | Homepage URL | `https://yourdomain.com` |
   | Authorization callback URL | `https://yourdomain.com/auth/github/callback` |

4. Click **"Register application"**
5. Note your **Client ID**
6. Click **"Generate a new client secret"** and note the **Client Secret** (shown only once)

{: .note }
> GitHub allows only one callback URL per OAuth app. For a second environment (e.g. local dev at `http://localhost:3033`), create a second OAuth App.

### Add to .env

```bash
SESSION_SECRET=<generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))">
GITHUB_CLIENT_ID=your_client_id_here
GITHUB_CLIENT_SECRET=your_client_secret_here
GITHUB_CALLBACK_URL=https://yourdomain.com/auth/github/callback
```

The login page will now show a **"Sign in with GitHub"** button.

### GitHub email requirements

GitHub OAuth only returns the user's email if at least one email address is set to **public** on their GitHub profile, or if the app requests the `user:email` scope (which taskpapr does). Users whose GitHub account has no accessible email will be rejected with a clear error message.

---

## OIDC / SSO setup (Authentik)

This works with any standards-compliant OIDC provider. The steps below are specific to **Authentik**.

### Create the Provider in Authentik

1. In your Authentik admin UI, go to **Applications → Providers**
2. Click **"Create"** → select **"OAuth2/OpenID Connect Provider"**
3. Fill in:

   | Field | Value |
   |---|---|
   | Name | `taskpapr` |
   | Authorization flow | Your standard login flow (e.g. `default-authentication-flow`) |
   | Client type | **Confidential** |
   | Redirect URIs/Origins | `https://yourdomain.com/auth/oidc/callback` |

4. Under **"Advanced protocol settings"**, ensure these scopes are available:
   - `openid`
   - `email`
   - `profile`

5. Click **"Finish"**
6. Note the **Client ID** and **Client Secret** from the provider detail page

### Create the Application in Authentik

1. Go to **Applications → Applications**
2. Click **"Create"**
3. Fill in:

   | Field | Value |
   |---|---|
   | Name | `taskpapr` |
   | Slug | `taskpapr` |
   | Provider | Select the provider you just created |

4. Optionally, under **"Policy / Group / User Bindings"**, bind a group to restrict which Authentik users can access taskpapr

5. Click **"Create"**

### Find your Issuer URL

The OIDC issuer URL for Authentik follows this pattern:

```
https://<your-authentik-domain>/application/o/<application-slug>/
```

For example, if your Authentik is at `auth.example.com` and the slug is `taskpapr`:

```
https://auth.example.com/application/o/taskpapr/
```

Verify this works by visiting:
```
https://auth.example.com/application/o/taskpapr/.well-known/openid-configuration
```

It should return a JSON document describing the OIDC endpoints.

### Add to .env

```bash
SESSION_SECRET=<generate with: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))">
OIDC_ISSUER=https://auth.example.com/application/o/taskpapr/
OIDC_CLIENT_ID=your_oidc_client_id
OIDC_CLIENT_SECRET=your_oidc_client_secret
OIDC_CALLBACK_URL=https://yourdomain.com/auth/oidc/callback
OIDC_TRUST_IDP=true
```

The login page will now show a **"Sign in with SSO"** button.

---

## Whitelist vs open registration

### Default behaviour

For **self-hosted installs without Stripe**, the whitelist is **on by default**. A user's email must be in the whitelist table before they can log in. The whitelist is managed from the admin UI at `/admin`.

For **hosted SaaS installs with `STRIPE_SECRET_KEY` set**, the whitelist is **off by default** — anyone who can authenticate (and who pays/trials) can sign up.

### Overriding with REQUIRE_WHITELIST

Set `REQUIRE_WHITELIST` explicitly to override the automatic behaviour:

| Value | Effect |
|---|---|
| `true` | Whitelist always enforced, regardless of Stripe configuration |
| `false` | Whitelist always off — open registration for all users |
| *(unset)* | Auto: off when Stripe is configured, on when it isn't |

### OIDC_TRUST_IDP

The `OIDC_TRUST_IDP` setting works independently of `REQUIRE_WHITELIST` and applies to OIDC logins only:

| `OIDC_TRUST_IDP` | Effect |
|---|---|
| `false` (default) | OIDC users must pass the whitelist check |
| `true` | Whitelist check skipped for OIDC logins — anyone authenticated by your IdP is trusted |

**Recommendation:** Set `OIDC_TRUST_IDP=true` when using Authentik. Authentik is your own IdP and you already control who has accounts there. It is redundant to maintain a second whitelist in taskpapr.

---

## First-user / admin bootstrap

The **very first user** to log in (regardless of provider) is automatically:
- Created without any whitelist check
- Granted admin privileges

This means you can deploy taskpapr with no whitelist entries, log in once, and then manage the whitelist from the `/admin` page.

{: .note }
> After first login, immediately go to `/admin` and add any other users' emails to the whitelist (if not using `OIDC_TRUST_IDP=true` or `REQUIRE_WHITELIST=false`).

---

## Complimentary accounts

Admins can grant any user **lifetime free access** regardless of subscription or trial status. This is managed from the `/admin` page — find the user in the users list and click **✓ Comp**.

Complimentary access:
- Bypasses all subscription and trial checks
- Is independent of Stripe — Stripe webhooks cannot affect it
- Can be revoked by clicking **✗ Revoke** in the admin users list
- Does not apply to admin accounts (admins are always free)

Complimentary users see a green `✓ comp` badge in the admin users list.

---

## Full .env reference

```bash
# Session
# Required for multi-user mode. Generate with:
# node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
SESSION_SECRET=change-me-to-a-long-random-string

# Server
PORT=3033
NODE_ENV=production

# Database
# SQLite (default): leave DATABASE_URL unset; DB_PATH sets the file location
DB_PATH=/opt/taskpapr/data/taskpapr.db
# PostgreSQL (optional): set DATABASE_URL to use PostgreSQL instead of SQLite
# Run `node db-migrate.js` once after setting this to create the schema
DATABASE_URL=postgresql://user:password@localhost:5432/taskpapr

# GitHub OAuth
# Omit or leave blank to disable GitHub login
GITHUB_CLIENT_ID=
GITHUB_CLIENT_SECRET=
GITHUB_CALLBACK_URL=https://yourdomain.com/auth/github/callback

# OIDC / SSO
# Omit or leave blank to disable SSO login
OIDC_ISSUER=https://auth.example.com/application/o/taskpapr/
OIDC_CLIENT_ID=
OIDC_CLIENT_SECRET=
OIDC_CALLBACK_URL=https://yourdomain.com/auth/oidc/callback

# true  = anyone who can log in via your IdP is trusted (recommended for Authentik)
# false = OIDC users must also be on the whitelist
OIDC_TRUST_IDP=true

# Whitelist enforcement
# true  = whitelist always on
# false = open registration (no whitelist)
# unset = auto (off when Stripe configured, on otherwise)
REQUIRE_WHITELIST=

# Telegram (optional)
# Get a bot token from @BotFather on Telegram
TELEGRAM_BOT_TOKEN=
TELEGRAM_NOTIFY_HOUR=8
TELEGRAM_BOT_USERNAME=
TELEGRAM_WEBHOOK_SECRET=
TELEGRAM_CHAT_ID=

# Stripe (optional — hosted/SaaS deployments only)
# Self-hosted installs without these vars are completely unaffected
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
STRIPE_SOLO_MONTHLY_PRICE_ID=
STRIPE_SOLO_ANNUAL_PRICE_ID=
APP_URL=https://yourdomain.com
STRIPE_TAX_ENABLED=false
```
