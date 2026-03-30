---
layout: default
title: Integrations
nav_order: 4
has_children: true
---

# Integrations
{: .no_toc }

Connect taskpapr to the tools you already use.
{: .fs-6 .fw-300 }

---

taskpapr exposes a standard REST API and a webhook endpoint, so it works with any HTTP-capable automation tool without custom plugins or connectors.

## Integration pages

| Page | What it covers |
|---|---|
| [n8n](n8n.md) | Workflow automation — email to task, GitHub issues, scheduled tasks, daily backups |

---

## Quick reference

Every integration uses the same authentication: an API key passed as a Bearer token.

```
Authorization: Bearer tp_<your-key>
```

Create keys at **/admin → API keys** (or Settings → API keys).

For a complete API reference, see [API & Webhooks](../features/api-and-webhooks.md).
