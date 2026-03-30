---
layout: default
title: n8n
parent: Integrations
nav_order: 1
---

# n8n
{: .no_toc }

Workflow automation for taskpapr — no custom node required.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Overview

[n8n](https://n8n.io) is an open-source workflow automation tool. It can push tasks into taskpapr in response to almost any external event — starred emails, GitHub issues, calendar reminders, Slack mentions, form submissions, and more.

No custom plugin or node is required. taskpapr exposes a standard REST API and webhook endpoint that n8n's built-in **HTTP Request** node handles natively.

---

## Prerequisites

1. A running taskpapr instance (local or on a server — must be reachable from n8n)
2. An API key — create one at **/admin → API keys** or **Settings → API keys**
3. n8n — self-hosted (`npx n8n`) or [n8n Cloud](https://n8n.io/cloud/)

---

## Connection patterns

There are two ways to interact with taskpapr from n8n:

| Pattern | When to use |
|---|---|
| `POST /api/webhook` | Push a task in response to an event — simplest, no tile ID needed |
| `POST /api/tasks`, `PATCH /api/tasks/:id`, etc. | Full REST control — when you need due dates, recurrence, notes |

Both require the same header:
```
Authorization: Bearer tp_<your-key>
```

### Authentication in n8n

**Recommended: create a Header Auth credential**

1. n8n → Credentials → Add credential → **Header Auth**
2. Name: `Authorization`
3. Value: `Bearer tp_<your-key>`
4. Select this credential in every HTTP Request node that calls taskpapr

---

## Webhook endpoint (simplest)

### HTTP Request node settings

| Field | Value |
|---|---|
| Method | `POST` |
| URL | `https://your-taskpapr-instance/api/webhook` |
| Authentication | Header Auth (see above) |
| Body Content Type | JSON |

### Supported webhook actions

| Action | Required fields | Optional |
|---|---|---|
| `add_task` | `title`, `tile` | `goal` |
| `complete` | `title` or `id` | — |
| `mark_wip` | `title` or `id` | — |
| `delete_task` | `title` or `id` | — |

`tile` and `title` (for find operations) use **partial, case-insensitive matching** — `"work"` matches a tile named `"Work stuff"`.

### Example body

```json
{
  "action": "add_task",
  "tile": "Work",
  "title": "{% raw %}{{ $json.subject }}{% endraw %}"
}
```

---

## Example workflows

### 1 — Starred Gmail → task

```
[Gmail Trigger]  label: STARRED
  ↓
[HTTP Request]
  POST /api/webhook
  { "action": "add_task", "tile": "Inbox", "title": "Email: {{ $json.subject }}" }
```

Add an **IF** node before the HTTP Request to filter noreply senders.

---

### 2 — GitHub issue assigned to me → task

```
[GitHub Trigger]  event: issues / action: assigned
  ↓
[IF]  assignee.login == "your-github-username"
  ↓ true
[HTTP Request]
  POST /api/webhook
  { "action": "add_task", "tile": "Work", "title": "GH#{{ $json.issue.number }}: {{ $json.issue.title }}" }
```

---

### 3 — Monday morning weekly review

```
[Schedule Trigger]  Every Monday at 08:30
  ↓
[HTTP Request]
  POST /api/webhook
  { "action": "add_task", "tile": "Work", "title": "Weekly review" }
```

{: .note }
> This is separate from taskpapr's built-in recurrence. Use n8n for calendar-driven prompts; use taskpapr's `recurrence` field for habit-style recurring tasks.

---

### 4 — Form submission → task

```
[Webhook Trigger]  (paste URL into Typeform/Tally integration)
  ↓
[HTTP Request]
  POST /api/webhook
  { "action": "add_task", "tile": "Client Requests", "title": "{{ $json.answers[0].text }}" }
```

---

### 5 — Daily automated board backup

```
[Schedule Trigger]  Daily at 02:00
  ↓
[HTTP Request]  GET /api/export  (Authorization: Bearer tp_...)
  ↓
[Write Binary File / Google Drive / S3 node]
  filename: taskpapr-{{ $now.format('YYYY-MM-DD') }}.json
```

---

### 6 — Todoist / Things migration (one-off import)

```
[Read file / Todoist API]
  ↓
[Code node]  transform to taskpapr import format:
  { "tiles": [{ "name": "Work", "tasks": [{ "title": "...", "status": "active" }] }] }
  ↓
[HTTP Request]  POST /api/import?mode=merge  (Authorization: Bearer tp_...)
```

---

## Email → task (full recipe)

Email parsing is intentionally not a native taskpapr feature — HTML email, threading, attachments, and MIME handling belong in middleware. n8n is the right layer.

### Option A — IMAP polling

Use the **Email (IMAP)** node to poll a dedicated capture inbox.

| n8n field | Use as |
|---|---|
| `$json.subject` | task title |
| `$json.text` | notes (plain-text body) |
| `$json.from.value[0].address` | sender (for filtering) |

**Limitations:** polls every N minutes (not real-time); requires an app password; use a dedicated address, not your main inbox.

### Option B — Webhook forwarding via Cloudmailin (recommended)

Configure Cloudmailin or Mailgun inbound to POST emails to an n8n **Webhook** node in real time.

**Cloudmailin setup:**
1. Create a free Cloudmailin address (e.g. `abc123@cloudmailin.net`)
2. Set the delivery target to your n8n Webhook node URL, format: **JSON**

**Cloudmailin field mappings:**

| Field | Use as |
|---|---|
| `$json.headers.subject` | task title |
| `$json.plain` | notes body |
| `$json.envelope.from` | sender address |

### Core flow

```
[Email trigger (IMAP or Webhook)]
  ↓
[Code node — clean subject]
  const raw = $input.first().json.subject
           || $input.first().json.headers.subject;
  const clean = (raw || 'Untitled email')
    .replace(/^(Re|Fwd?|FW|RE):\s*/i, '')
    .replace(/\s+/g, ' ').trim().substring(0, 120);
  return [{ json: { title: clean } }];
  ↓
[HTTP Request]
  POST /api/webhook
  { "action": "add_task", "tile": "Inbox", "title": "{{ $json.title }}" }
```

### Optional: write email body to task notes

The `add_task` webhook action doesn't accept a `notes` field. Use a second request:

```
[HTTP Request #1]  POST /api/webhook  → returns { "task": { "id": 42 } }
  ↓
[HTTP Request #2]  PATCH /api/tasks/{{ $('Create task').item.json.task.id }}
  { "notes": "{{ $('Code').item.json.body.substring(0, 2000) }}" }
```

### Filtering — avoid noise from automated senders

```
[IF node]
  $json.from does NOT contain "noreply"
  AND does NOT contain "no-reply"
  ↓ true branch only
[HTTP Request → taskpapr]
```

Route to different tiles by sender domain:

```javascript
// Code node: choose tile by sender domain
const from = $input.first().json.envelope?.from || $input.first().json.from || '';
const domain = from.split('@')[1] || '';
let tile = 'Inbox';
if (domain.includes('github.com'))    tile = 'Work';
if (domain.includes('mycompany.com')) tile = 'Work';
if (domain.includes('bank'))          tile = 'Finance';
return [{ json: { ...$input.first().json, tile } }];
```

### User tip

Save your Cloudmailin address as a contact named `📥 taskpapr`. Forwarding an email becomes: forward → type "taskpapr" → send. Three gestures.

---

## Full REST API (for advanced flows)

### Create a task with a due date

```
POST /api/tasks
{ "title": "Pay VAT return", "column_id": 3 }
  ↓
PATCH /api/tasks/{{ $json.id }}
{ "next_due": "2026-01-31", "recurrence": "3m" }
```

### Complete a task by ID

```
PATCH /api/tasks/42
{ "status": "done" }
```

### Find a tile ID by name

```javascript
// Code node
const cols = $('Get Columns').all();
const tile = cols[0].json.find(c => c.name.toLowerCase().includes('work'));
return [{ json: { tile_id: tile.id } }];
```

---

## Error handling

The webhook returns structured errors you can handle in n8n:

| Situation | Status | Response |
|---|---|---|
| Missing or invalid API key | 401 | `{ "error": "unauthorized" }` |
| Tile not found | 404 | `{ "error": "tile not found: \"X\"", "available": ["Work", …] }` |
| Task not found | 404 | `{ "error": "task not found: \"X\"" }` |
| Unknown action | 400 | `{ "error": "unknown action: \"X\"", "supported": […] }` |

Use **Continue on Fail** + an **IF** node to handle failures gracefully, or attach an **Error Trigger** workflow.

---

## Useful n8n expressions

```javascript
// Today's date as YYYY-MM-DD
{{ $now.format('YYYY-MM-DD') }}

// Truncate a long subject to 80 chars
{{ $json.subject.substring(0, 80) }}

// Strip Re: / Fwd: prefixes
{{ $json.subject.replace(/^(Re|Fwd|FW|RE):\s*/i, '') }}
```

---

## Self-hosting n8n alongside taskpapr

If taskpapr runs on a Linux server and you want n8n on the same machine:

```bash
npm install -g n8n
```

Create `/etc/systemd/system/n8n.service`:

```ini
[Unit]
Description=n8n workflow automation
After=network.target

[Service]
Type=simple
User=taskpapr
WorkingDirectory=/opt/n8n
Environment=N8N_PORT=5678
Environment=WEBHOOK_URL=https://n8n.yourdomain.com/
ExecStart=/usr/bin/n8n start
Restart=always

[Install]
WantedBy=multi-user.target
```

Add a Traefik route for `n8n.yourdomain.com → localhost:5678`. taskpapr and n8n can then call each other via `http://localhost:3033` and `http://localhost:5678` without leaving the server.

---

## Importable workflow

A complete exportable n8n workflow for the email-to-task recipe (Cloudmailin trigger, sender filtering, optional body → notes) is available in the app repo at `docs/n8n-email-workflow.json`.

**After importing:**
1. Update both HTTP Request node URLs to your taskpapr instance
2. Replace `tp_YOUR_API_KEY_HERE` with your real API key
3. Paste your n8n Webhook URL into Cloudmailin as the delivery target
