---
layout: default
title: Features
nav_order: 3
has_children: true
---

# Features
{: .no_toc }

Everything taskpapr can do — from the core board to integrations and automation.
{: .fs-6 .fw-300 }

---

taskpapr is a minimal, paper-inspired task board that runs locally or on a server you control. It has no accounts to create, no cloud lock-in, and no noise. Below is a map of everything it can do.

## Feature pages

| Page | What it covers |
|---|---|
| [The Board](the-board.md) | Infinite canvas, tiles, tasks, WIP state, search, keyboard shortcuts |
| [Recurring Tasks](recurring-tasks.md) | Due dates, spinning plates urgency heat, dormancy, task rot, snooze |
| [Goals](goals.md) | Outcome tracking, goal smart-tiles, assigning tasks to goals |
| [Telegram](telegram.md) | Daily digest, self-service connect flow, quick-capture from chat |
| [API & Webhooks](api-and-webhooks.md) | REST API reference, webhook actions, n8n automation examples |
| [MCP Server](mcp.md) | AI assistant integration — Claude Desktop, Cline, 9 available tools |
| [Import & Export](import-export.md) | Backup, restore, merge vs replace, curl examples |

---

## At a glance

**The board** is an infinite canvas. Tiles (columns) sit wherever you put them. Tasks live inside tiles. Nothing is forced into a grid.

**Recurring tasks** have a visual urgency system — amber → orange → red as a due date approaches. One-off tasks have a separate rot system — they quietly age to parchment if left untouched. Both are purely visual; no notifications, no badges.

**Goals** are outcomes, not projects. Assign tasks to a goal and see them collected in a smart-tile on the canvas. Track progress by task count.

**Telegram** sends a daily digest of tasks due today and tomorrow. You can also send plain text to the bot to create tasks instantly from your phone.

**The API** is a standard REST API with Bearer token auth. The webhook endpoint accepts four actions (`add_task`, `complete`, `mark_wip`, `delete_task`) and works with n8n, Zapier, Make, or any HTTP client.

**The MCP server** exposes your board as tools for Claude Desktop, Cline, or any MCP-compatible AI assistant. Ask it what's on your board, add tasks, mark things done — in plain language.

**Export/import** produces a portable JSON file. Import in merge or replace mode. Use it for backups, migrating between instances, or loading demo data.

---

{: .note }
> Want to suggest a feature? [Open an issue on GitHub](https://github.com/taskpapr/taskpapr).
