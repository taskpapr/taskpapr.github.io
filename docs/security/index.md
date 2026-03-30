---
layout: default
title: Security
nav_order: 5
has_children: true
---

# Security
{: .no_toc }

How taskpapr is hardened — what's built in, what you configure, and the recommended infrastructure pattern.
{: .fs-6 .fw-300 }

---

taskpapr is designed to be self-hosted on infrastructure you control. This section covers the security posture of the application itself and the recommended deployment architecture for a public-facing instance.

## Pages

| Page | What it covers |
|---|---|
| [Security Reference](security-reference.md) | Built-in protections, v0.35.0 hardening, Cloudflare + Traefik infrastructure pattern, ongoing hygiene |

---

## Quick summary

**You don't need to configure anything for a private/personal deployment.** The defaults are safe.

For a **public-facing deployment** (open to the internet), the recommended pattern is:

```
Cloudflare (free) → Traefik (rate limiting + HSTS) → taskpapr (app-layer hardening)
```

Three independent layers of rate limiting. Your VPS IP is never exposed. See the [Security Reference](security-reference.md) for the full setup.
