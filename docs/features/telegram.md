---
layout: default
title: Telegram
parent: Features
nav_order: 4
---

# Telegram
{: .no_toc }

Daily digest and quick-capture from your phone.
{: .fs-6 .fw-300 }

---

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## What you get

Once connected, taskpapr sends you a daily digest each morning via Telegram:

```
📋 taskpapr reminder

Due today (2026-03-02):
  • Pay credit card [Personal]
  • Weekly review [Work]

Due tomorrow (2026-03-03):
  • Call accountant [Work]
```

The digest includes tasks with a `next_due` date set to today or tomorrow. Dormant tasks and done tasks are excluded.

You can also send plain text to the bot to **quick-capture** tasks from anywhere — your phone, desktop, wherever Telegram is open.

---

## Prerequisites

1. A Telegram account
2. taskpapr running with `TELEGRAM_BOT_TOKEN` set (see [Deployment](../deployment.md))
3. `TELEGRAM_BOT_USERNAME` set (so the connect flow can show you the correct bot to message)

---

## Connecting your account (self-service)

You connect your own Telegram account from the taskpapr Settings page. No token is ever exposed to you — the flow uses a short-lived code.

1. Open taskpapr → click your user avatar → **⚙ Settings**
2. Scroll to the **Telegram** section
3. Click **Connect Telegram**
4. taskpapr shows you a 6-character code and a `/start` command, e.g.:
   ```
   /start ABC123
   ```
5. Open Telegram, find your bot (the username shown on screen), and send that exact `/start ABC123` command
6. The bot replies with a confirmation message
7. The Settings page detects the connection within a few seconds and switches to the connected state

The code expires after 10 minutes and can only be used once.

---

## Sending a test notification

Once connected, click **Send test notification** in Settings → Telegram. You should receive a message within a few seconds.

If you have no tasks due today or tomorrow, the message will say so. That's correct behaviour, not an error.

---

## Daily digest timing

The digest is sent once per day at the hour set by `TELEGRAM_NOTIFY_HOUR` (default: `8` = 8am server local time).

It also runs ~10 seconds after the server starts — useful if you restart the server after 8am and want the digest immediately.

---

## Quick-capture

Send any plain text message to the bot to create a task instantly.

**Default behaviour:** the task is created in your capture tile. If you haven't set a capture tile, taskpapr uses an "Inbox" tile (creating it if it doesn't exist).

**Routing to a specific tile:** prefix your message with the tile name followed by a colon:

```
Work: Review PR #42
```

The tile name is matched case-insensitively. A partial match is fine — `work` matches `Work`.

If the named tile isn't found, the bot replies with the available tile names so you can correct the prefix.

**Bot reply on success:**
```
✅ Added to Work: Review PR #42
```

### Setting your capture tile

Open Settings → Telegram → **Quick-capture tile**. Type the name of the tile you want new bot messages to land in. Save.

---

## Per-user chat IDs (multi-user setup)

Each user connects their own Telegram account via the self-service flow above. The bot then sends each user only their own tasks.

The server-level `TELEGRAM_CHAT_ID` in `.env` acts as a fallback for users who haven't connected. For a single-user setup this is the only configuration needed (no self-service flow required).

All users message the **same bot** — the bot routes messages to different chat IDs. You don't need a separate bot per user.

---

## Disconnecting

Open Settings → Telegram → **Disconnect**. This clears your chat ID from your account. The bot will stop sending you messages.

---

## Troubleshooting

**"TELEGRAM_BOT_TOKEN or TELEGRAM_CHAT_ID not set — notifications disabled"**
The server started without the required env vars. Add them to `.env` and restart.

**The connect flow shows a code but the Settings page never updates**
Make sure you sent `/start ABC123` (including `/start`, not just the code) to the correct bot. Check `TELEGRAM_BOT_USERNAME` is set correctly in `.env`.

**The bot doesn't respond to messages**
The webhook may not be registered. Check that `TELEGRAM_BOT_TOKEN` is valid and that the server is reachable from the internet (Telegram must be able to POST to your instance).

**"403 Forbidden" from the Telegram API**
The user hasn't messaged the bot yet. Telegram bots can only send to users who have initiated a conversation first. Send `/start` to the bot, then retry.

**Wrong notification time**
`TELEGRAM_NOTIFY_HOUR` uses the server's local time. If your server is UTC and you want 8am London time (UTC+1), set `TELEGRAM_NOTIFY_HOUR=7` in winter or `8` in summer, or set your server's timezone correctly (`timedatectl` on Linux).

**"No tasks due today or tomorrow"**
Working correctly. No message is sent when there's nothing due. Add a task with today's date and click Send test notification to verify the flow is working.
