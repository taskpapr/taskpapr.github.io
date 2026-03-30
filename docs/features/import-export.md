---
layout: default
title: Import & Export
parent: Features
nav_order: 7
---

# Import & Export
{: .no_toc }

Portable JSON backups — for peace of mind, migration, and demo data.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Export

Export produces a complete JSON snapshot of your board: all tiles, tasks, and goals.

### What's included

- All tiles (name, position, size, colour, scale, hidden state)
- All tasks (title, status, notes, due date, recurrence, colour, goal assignment, rot settings)
- All goals (title, notes)

Tasks reference goals by title, not internal ID — so exports are portable across instances.

### How to export

**From the UI:** User avatar → **⚙ Settings** → **Export** → Download. The browser auto-downloads a file named `taskpapr-export-YYYY-MM-DD.json`.

**Via the API:**
```bash
curl -s https://your-instance/api/export \
  -H "Authorization: Bearer tp_..." \
  > taskpapr-backup.json
```

### Export format

```json
{
  "version": "1",
  "exported_at": "2026-03-30T10:00:00.000Z",
  "taskpapr_version": "0.33.0",
  "goals": [
    { "title": "Launch MVP", "notes": null, "position": 1 }
  ],
  "tiles": [
    {
      "name": "Work",
      "x": 40, "y": 40, "width": 260,
      "color": null,
      "position": 1,
      "tasks": [
        {
          "title": "Write proposal",
          "status": "active",
          "position": 0,
          "goal": "Launch MVP",
          "notes": null,
          "created_at": "2026-02-27 09:00:00"
        }
      ]
    }
  ]
}
```

---

## Import

Import reads a taskpapr JSON export file and adds it to your board.

### Merge vs replace

| Mode | Behaviour |
|---|---|
| **merge** (default) | Adds tiles and tasks alongside your existing data. If a tile with the same name already exists, tasks are added to it — no duplicate tiles created. Safe to re-import the same file. |
| **replace** | Deletes all your existing tiles, tasks, and goals first, then imports the file. Use for a full restore or loading demo data on a fresh board. |

{: .warning }
> **Replace mode is destructive.** Export your current board first if you want a recovery point.

### How to import

**From the UI:** User avatar → **⚙ Settings** → **Import** → choose a file → select Merge or Replace → Import.

**Via the API — merge:**
```bash
curl -X POST 'https://your-instance/api/import?mode=merge' \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d @taskpapr-backup.json
```

**Via the API — replace (full restore):**
```bash
curl -X POST 'https://your-instance/api/import?mode=replace' \
  -H "Authorization: Bearer tp_..." \
  -H "Content-Type: application/json" \
  -d @taskpapr-backup.json
```

### Import response

```json
{
  "ok": true,
  "mode": "merge",
  "imported": {
    "goals": 1,
    "tiles": 2,
    "tasks": 12,
    "skipped": 0
  }
}
```

`skipped` counts items that were silently dropped due to missing required fields (e.g. a task with no title).

---

## Use cases

### Regular backups

Run the export curl command from a cron job:

```bash
# Daily backup at 2am
0 2 * * * curl -s https://your-instance/api/export \
  -H "Authorization: Bearer tp_..." \
  > /opt/backups/taskpapr-$(date +\%Y\%m\%d).json
```

Or automate with n8n: Schedule trigger → HTTP Request (`GET /api/export`) → save to disk or send to S3.

### Migrating between instances

1. Export from the old instance: `GET /api/export`
2. Import to the new instance: `POST /api/import?mode=replace`

Goals are referenced by title so they survive the move correctly.

### Loading demo data

Prepare a curated JSON file with representative tiles and tasks. Import it with `mode=replace` to reset a demo or staging instance to a known state.

### Moving between users

The export format is per-user. To copy one user's board to another:
1. Log in as the source user, export
2. Log in as the destination user, import with `mode=replace`
