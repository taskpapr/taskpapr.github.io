---
layout: default
title: MCP Server
parent: Features
nav_order: 6
---

# MCP Server
{: .no_toc }

Talk to your task board from any AI assistant.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What MCP is

MCP (Model Context Protocol) is a standard that lets AI assistants — Claude, Cline, and others — connect to external tools and data sources. The taskpapr MCP server exposes your board as a set of tools that any MCP-compatible client can call.

Once connected, you can ask things like:

> "What's on my board today?"
> "Add 'Review PR #42' to my Work tile"
> "Mark the proposal task as WIP"
> "What tasks are linked to the Launch MVP goal?"

---

## Prerequisites

1. taskpapr running (locally or on a server)
2. Node.js 22.5+ installed on the machine running the AI client
3. An API key — create one at **/admin → API keys**

---

## Available tools

**Reading the board**

| Tool | What it does |
|---|---|
| `get_board_summary` | Full board overview — all tiles, active/WIP tasks, goals |
| `list_tiles` | All tiles with task counts |
| `list_tasks` | Tasks, filterable by tile name and/or status |
| `search_tasks` | Find tasks by title (and optionally notes) substring, filterable by tile/goal/status — the safe way to locate a task before updating or deleting it |

**Adding and changing tasks**

| Tool | What it does |
|---|---|
| `add_task` | Add a task to a named tile. Blocks exact-title duplicates in the same tile by default (pass `allow_duplicate: true` to add one anyway) |
| `update_task` | Rename a task, move it to a different tile, assign or clear its goal, change its status, or replace its notes — one partial-update tool rather than several single-purpose ones |
| `append_task_note` | Append text to a task's notes without overwriting what's already there. Appended text is marked with a provenance line (`— via MCP, YYYY-MM-DD`) so it's clearly distinguishable from notes you typed yourself |
| `complete_task` | Mark a task done (by id or title match) |
| `mark_wip` | Mark a task as WIP (by id or title match) |
| `snooze_task` | Hide a task from the active board until later. With no arguments it's the same fixed 24h snooze as the app's Snooze button; pass `hours`/`days` or an explicit `until` date for a custom duration |
| `delete_task` | **Permanently** delete a task. An unambiguous `id` deletes immediately; a `title` match requires `confirm: true` on a second call, and is rejected outright if the title matches more than one task |

**Today list**

| Tool | What it does |
|---|---|
| `list_today_tasks` | List tasks flagged for Today, in the same order the Today tile shows them |
| `add_task_to_today` | Add an existing task to Today (appended to the end) |
| `remove_task_from_today` | Remove a task from Today |
| `reorder_today_tasks` | Set the full order of everything currently in Today |

**Goals**

| Tool | What it does |
|---|---|
| `list_goals` | All goals with task counts |
| `add_goal` | Create a new goal |

Every tool that acts on a specific task accepts either an `id` (unambiguous, preferred when known) or a `title_match`/`title` substring. A title match that resolves to more than one task is never guessed at — the call returns the candidate matches instead of picking one, so you can retry with a specific id.

---

## Claude Desktop setup

Claude Desktop uses the stdio MCP transport. Edit its config file:

**Config file location (macOS):**
```
~/Library/Application Support/Claude/claude_desktop_config.json
```

**Local taskpapr (macOS):**
```json
{
  "mcpServers": {
    "taskpapr": {
      "command": "node",
      "args": ["/path/to/taskpapr/mcp/server.js"],
      "env": {
        "TASKPAPR_URL": "http://localhost:3033",
        "TASKPAPR_API_KEY": "tp_your_key_here"
      }
    }
  }
}
```

**Remote instance:**
```json
{
  "mcpServers": {
    "taskpapr": {
      "command": "node",
      "args": ["/path/to/taskpapr/mcp/server.js"],
      "env": {
        "TASKPAPR_URL": "https://your-instance.example.com",
        "TASKPAPR_API_KEY": "tp_your_key_here"
      }
    }
  }
}
```

After saving, restart Claude Desktop. The taskpapr tools will appear in the tools list.

---

## Cline (VS Code) setup

Add to your Cline MCP settings:

```json
{
  "taskpapr": {
    "command": "node",
    "args": ["/path/to/taskpapr/mcp/server.js"],
    "env": {
      "TASKPAPR_URL": "http://localhost:3033",
      "TASKPAPR_API_KEY": "tp_your_key_here"
    }
  }
}
```

---

## Environment variables

| Variable | Required | Default | Description |
|---|---|---|---|
| `TASKPAPR_API_KEY` | Yes | — | API key from `/admin` |
| `TASKPAPR_URL` | No | `http://localhost:3033` | Base URL of your taskpapr instance |

---

## Example prompts

Once connected, the AI can answer questions and take actions in plain language:

**Read the board:**
- "What tasks are on my board?"
- "What's currently in progress?"
- "Do I have anything linked to the Launch MVP goal?"

**Add tasks:**
- "Add 'Send invoice to Acme' to my Work tile"
- "Add 'Book dentist' to Personal and link it to the Health goal"

**Update tasks:**
- "Mark the proposal task as WIP"
- "Mark 'Send invoice' as done"
- "Move the dentist task to Personal and link it to the Health goal"
- "Add a note to the proposal task: waiting on legal review"
- "Snooze the tax return task for 3 days"

**Today list:**
- "What's in my Today list?"
- "Add the proposal task to Today"
- "Reorder Today so the invoice task is first"

**Find and clean up:**
- "Search for anything mentioning 'invoice'"
- "Delete the 'old draft' task" *(asks you to confirm before it deletes anything by title)*

**Manage goals:**
- "Show me my goals and how many tasks each has"
- "Create a new goal called 'Q2 Planning'"

---

## Troubleshooting

**"TASKPAPR_API_KEY is not set"**
The env var is missing from the MCP config. Check your `claude_desktop_config.json` or Cline settings.

**"taskpapr API error 401"**
The API key is invalid or has been revoked. Create a new one at `/admin → API keys`.

**"Connection refused"**
taskpapr isn't running. Start it with `npm start`, or check the systemd/launchd service.

**Tools don't appear in Claude Desktop**
Restart Claude Desktop after editing the config. Check the config file is valid JSON.
