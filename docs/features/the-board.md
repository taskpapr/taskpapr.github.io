---
layout: default
title: The Board
parent: Features
nav_order: 1
---

# The Board
{: .no_toc }

The infinite canvas at the heart of taskpapr.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Canvas

The board is an infinite canvas. There are no pages, no fixed grid, and no forced layout. Tiles sit wherever you put them.

### Navigating

| Action | How |
|---|---|
| Pan | Click and drag on empty canvas |
| Zoom in / out | Scroll wheel, or `+` / `-` keys |
| Reset zoom to 100% | `Home` key |
| Zoom indicator | Bottom-left corner; shows current zoom % |

The canvas position and zoom level are saved per-device in `localStorage` and restored on reload.

### Off-screen beacons

When tiles exist outside the visible area, pill-shaped indicators appear at the edges of the viewport — top, bottom, left, right — showing how many tiles are off-screen in that direction. Click a beacon to pan to the nearest off-screen tile in that direction.

Beacons pulse gently and disappear as tiles come into view.

---

## Tiles

Tiles are columns. Each tile holds a list of tasks. Tiles can be freely positioned anywhere on the canvas.

### Adding a tile

Click **+ Add tile** in the header.

### Renaming a tile

Click the tile header text to edit it inline.

### Moving a tile

Drag the tile by its header. On mobile / iPad, press and hold the header for ~400ms to initiate drag.

### Resizing a tile

Drag the resize handle in the bottom-right corner of the tile.

### Per-tile colour

Click the 🎨 icon on the tile header (visible on hover) to open the palette picker. Choose from 8 colours, or clear back to the default parchment.

### Per-tile scale

Use the **−** / **+** buttons on the tile header (visible on hover) to shrink or enlarge the tile's content in 0.1× steps. Range: 0.5× – 2.0×. Default: 1.0×.

{: .note }
> Scaling affects the text and tasks inside the tile, not the tile's canvas footprint.

### Deleting a tile

Click the **✕** icon on the tile header (visible on hover). This deletes the tile and all its tasks permanently.

### Hidden tiles (Someday / Maybe)

Any tile can be hidden from the main board. Hidden tiles are for tasks you want to keep but don't want cluttering your everyday view — a Someday/Maybe list.

- **Hide a tile:** Click the 🙈 icon on the tile header (visible on hover)
- **Reveal hidden tiles:** Click the **🗃 N** button in the header (only visible when hidden tiles exist). Hidden tiles are shown with a dashed border and hatched header.
- **Unhide a tile:** Click the 🙉 icon on the tile header while it's revealed.

---

## Tasks

### Adding a task

Click **+ Add task** at the bottom of any tile, type, and press Enter. The task is added immediately.

### Checking off a task

Click the checkbox on the left of the task. The task stays visible with strikethrough — it doesn't disappear. This is intentional: the visual record of done work is the progress signal.

Checking off a task does not delete it. Use **Clear done** (see below) when you're ready to clear a tile.

### Inline editing

Double-click a task title to edit it in place.

### Task detail panel

Click a task title (single click) to open the detail panel on the right. The panel contains:
- Editable title and notes (auto-saved, debounced)
- Notes render as live Markdown preview
- Due date, recurrence interval, visibility window
- Goal assignment
- Colour picker
- Rot settings (see [Recurring Tasks](recurring-tasks.md))
- Touched / Created metadata
- Action buttons: Find on board, Park, Touch

### Dragging tasks

Drag tasks to reorder within a tile, or drag them to a different tile entirely.

### Clearing done tasks

- **Per tile:** Click the **Clear done** button on the tile header (visible when done tasks exist). Tasks animate out with a fade-and-collapse effect.
- **Global:** Click the **Clear done** button in the main header to clear done tasks across all tiles.

---

## WIP state

Mark a task as **Work In Progress** to give it a visual amber left stripe and a WIP badge.

- **Mark WIP:** Click the left stripe of any active task, or use the context menu → **Mark WIP**
- **Clear WIP:** Click the **WIP** badge on the task

WIP is a signal, not a filter. WIP tasks remain in place on the board.

---

## Context menu

Right-click any task (or click the **⋯** button) to open the context menu:

| Option | What it does |
|---|---|
| Properties | Opens the task detail panel |
| Mark WIP | Sets WIP state |
| Set colour… | Opens the 8-colour palette picker |
| Assign Goal | Links the task to a goal |
| Touch | Resets the rot clock (acknowledges you've seen this task) |
| Snooze 24h | Hides the task for 24 hours without changing its due date |
| Park | Moves the task to a hidden tile (creates one if none exists) |
| Delete | Permanently deletes the task |

---

## Search

Press **⌘K** (or **Ctrl+K**) to open the command palette search.

- Searches all tasks, including dormant ones
- Results are score-ranked
- Matching text is highlighted in yellow
- Click a result to open the task detail panel; if the task is dormant, it is auto-revealed in its tile

Click the **🔍** button in the header to open search with the mouse.

---

## Keyboard shortcuts

| Shortcut | Action |
|---|---|
| `⌘K` / `Ctrl+K` | Open search |
| `+` | Zoom in |
| `-` | Zoom out |
| `Home` | Reset zoom to 100% |
| `G` | Toggle goal smart-tiles |
| `Escape` | Close open panels / dismiss overlays |

---

## Canvas bookmarks

{: .label .label-yellow }
Coming in v0.34.0

Named saved views. A bookmark stores the current canvas position and zoom level. Click a bookmark to animate back to that view instantly.

Useful for maintaining separate "zones" on the canvas — e.g. jump between a Work area and a Personal area with one click.
