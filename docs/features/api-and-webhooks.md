---
layout: default
title: API & Webhooks
parent: Features
nav_order: 5
---

# API & Webhooks
{: .no_toc }

Automate and integrate taskpapr with any tool.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Authentication

All API endpoints require authentication — either a browser session cookie or an API key.

### API keys (Bearer token)

Create keys in the Admin UI: **/admin → API keys**, or via the API:

```bash
POST /api/admin/api-keys
{"name": "my key"}
```

Use in requests:

```
Authorization: Bearer tp_<your key>
```

Keys are one-way hashed — the raw value is shown **once** on creation. Store it securely.

To revoke a key: Admin UI → API keys → Delete, or `DELETE /api/admin/api-keys/:id`.

{: .note }
> The webhook endpoint requires an API key. Session cookies are intentionally not accepted for webhook calls.

---

## Webhook

The webhook endpoint is designed for push-based automation from tools like n8n, Zapier, Make, or IFTTT.

### Endpoint

```
POST /api/webhook
Authorization: Bearer tp_<api-key>
Content-Type: application/json
```

### Supported actions

| Action | Required fields | Optional fields |
|---|---|---|
| `add_task` | `title`, `tile` | `goal` |
| `complete` | `title` or `id` | — |
| `mark_wip` | `title` or `id` | — |
| `delete_task` | `title` or `id` | — |

**Matching rules:**
- `tile` — partial, case-insensitive match against tile names (`"work"` matches `"Work"`)
- `title` (for find operations) — partial, case-insensitive match against task titles
- `goal` — partial, case-insensitive match against goal titles
- `id` — exact task id (preferred over title when known)

### Examples

**Add a task:**
```bash
curl -X POST https://your-instance/api/webhook \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d '{"action":"add_task","title":"Review PR #42","tile":"Work"}'
```

**Add a task and link to a goal:**
```bash
curl -X POST https://your-instance/api/webhook \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d '{"action":"add_task","title":"Write draft","tile":"Work","goal":"Launch MVP"}'
```

**Mark a task done:**
```bash
curl -X POST https://your-instance/api/webhook \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d '{"action":"complete","title":"Review PR"}'
```

**Mark a task WIP:**
```bash
curl -X POST https://your-instance/api/webhook \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d '{"action":"mark_wip","id":42}'
```

### Tile not found response

If the tile isn't found, the API returns a helpful 404:

```json
{
  "error": "tile not found: \"Inbox\"",
  "available": ["Work", "Personal", "Errands", "Side Business"]
}
```

---

## n8n automation

n8n is a good fit for taskpapr automation — self-hostable, visual, and it speaks REST natively.

### Setup

Use the **HTTP Request** node:

| Field | Value |
|---|---|
| Method | `POST` |
| URL | `https://your-instance/api/webhook` |
| Authentication | Header Auth: `Authorization: Bearer tp_...` |
| Body Content Type | JSON |
| JSON body | `{"action": "add_task", "title": "...", "tile": "..."}` |

### Example recipes

**Starred email → task in Inbox**
Trigger: Email trigger (IMAP) → filter for starred/flagged → HTTP Request: `add_task` with email subject as title, tile `Inbox`.

**GitHub issue assigned to me → task in Work tile**
Trigger: GitHub trigger (issue assigned) → HTTP Request: `add_task` with issue title, tile `Work`.

**Weekly recurring trigger → create a task**
Trigger: Schedule (every Monday 8am) → HTTP Request: `add_task` with title `Weekly review`, tile `Work`.

---

## REST API reference

### `/api/me`

**GET** — Current authenticated user (or the synthetic local user in single-user mode).

The response always includes **`id`**, **`display_name`**, **`avatar_url`** (may be `null`), **`email`**, **`is_admin`**, and **`version`** (app version from `package.json`).

Additional fields are merged from the server for every mode:

| Field | Meaning |
|---|---|
| `single_user` | `true` when the instance runs without login providers; `false` when multi-user auth is enabled |
| `debug_date` | Admin debug date override, or `null` |
| `telegram_chat_id` | User’s linked Telegram chat, env fallback, or `null` |
| `telegram_capture_tile` | Default capture tile name, or `null` |
| `stripe_configured` | Whether Stripe env is present (`true` / `false`) |
| `subscription_status` | Stripe subscription state (e.g. `trialing`, `active`) |
| `subscription_tier` | Tier name or `null` |
| `trial_ends_at` | Trial end date string or `null` |
| `trial_days_left` | Integer days remaining on trial, or `null` when not trialing |
| `has_billing_account` | Whether a Stripe customer id exists |

**Single-user mode** (no login) — example shape:

```json
{
  "id": 1,
  "display_name": "James Freeman",
  "avatar_url": null,
  "email": null,
  "is_admin": 1,
  "single_user": true,
  "version": "0.44.5",
  "debug_date": null,
  "telegram_chat_id": null,
  "telegram_capture_tile": null,
  "stripe_configured": false,
  "subscription_status": "trialing",
  "subscription_tier": null,
  "trial_ends_at": null,
  "trial_days_left": null,
  "has_billing_account": false
}
```

**Multi-user mode** (after GitHub, Google, or OIDC login) — same base keys; `single_user` is `false`, and billing/Telegram fields reflect the signed-in user. Values vary with your Stripe and Telegram configuration.

---

### Tiles

#### `GET /api/columns`
All tiles for the current user, ordered by position.

```json
[{ "id": 1, "name": "Work", "x": 40, "y": 40, "width": 260, "color": null, "position": 1 }]
```

#### `POST /api/columns`
Create a tile. `name` required; `x`, `y`, `width`, `color` optional.

#### `PATCH /api/columns/:id`
Update tile properties. Send only the fields you want to change: `name`, `x`, `y`, `width`, `color`, `position`, `scale`.

#### `DELETE /api/columns/:id`
Delete a tile and all its tasks.

---

### Tasks

#### `GET /api/tasks`
All tasks for the current user.

```json
[{
  "id": 1,
  "title": "Write proposal",
  "status": "active",
  "column_id": 1,
  "position": 0,
  "goal_id": null,
  "created_at": "2026-02-27 09:00:00",
  "updated_at": "2026-02-27 09:00:00"
}]
```

**Task statuses:** `active` | `wip` | `done` | `dormant`

#### `POST /api/tasks`
Create a task. `title` and `column_id` required.

#### `PATCH /api/tasks/:id`
Update a task. All fields optional: `title`, `status`, `goal_id`, `position`, `column_id`, `color`, `next_due`, `recurrence`, `visibility_days`.

#### `DELETE /api/tasks/:id`
Delete a single task.

#### `DELETE /api/tasks?column_id=<id>`
Delete all `done` tasks in a tile.

#### `DELETE /api/tasks`
Delete all `done` tasks across all tiles.

#### `POST /api/tasks/reorder`
Reorder and/or move tasks between tiles in one transaction.

```json
[
  { "id": 3, "position": 0, "column_id": 1 },
  { "id": 1, "position": 1, "column_id": 1 }
]
```

#### `POST /api/tasks/:id/ack`
Reset the rot clock (`last_acknowledged_at`) without any other change.

#### `POST /api/tasks/:id/snooze`
Snooze a task for 24 hours.

#### `POST /api/tasks/:id/park`
Move a task to a hidden tile (creates one if none exists).

---

### Goals

#### `GET /api/goals`
All goals.

```json
[{ "id": 1, "title": "Launch MVP", "notes": null, "position": 1 }]
```

#### `POST /api/goals`
Create a goal. Body: `{ "title": "...", "notes": "..." }`

#### `PATCH /api/goals/:id`
Update a goal. Body: `{ "title": "...", "notes": "..." }`

#### `DELETE /api/goals/:id`
Delete a goal. Linked tasks have `goal_id` set to null.

---

### Export / Import

#### `GET /api/export`
Download a complete JSON backup. See [Import & Export](import-export.md) for format details.

#### `POST /api/import?mode=merge|replace`
Import a JSON backup. See [Import & Export](import-export.md) for merge vs replace behaviour.

---

### Admin endpoints

All require `is_admin: true`.

| Endpoint | Method | Description |
|---|---|---|
| `/api/admin/users` | GET | List all registered users |
| `/api/admin/whitelist` | GET | List the invite whitelist |
| `/api/admin/whitelist` | POST | Add email to whitelist |
| `/api/admin/whitelist/:id` | DELETE | Remove from whitelist |
| `/api/admin/api-keys` | GET | List API keys (no raw values) |
| `/api/admin/api-keys` | POST | Create an API key |
| `/api/admin/api-keys/:id` | DELETE | Revoke an API key |
