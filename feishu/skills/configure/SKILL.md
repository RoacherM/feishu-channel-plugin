---
name: configure
description: Set up the Feishu channel — save the App ID and App Secret, review access policy. Use when the user pastes Feishu credentials, asks to configure Feishu, asks "how do I set this up" or "who can reach me," or wants to check channel status.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Bash(ls *)
  - Bash(mkdir *)
---

# /feishu:configure — Feishu Channel Setup

Write the App ID and App Secret to `~/.claude/channels/feishu/.env` and
orient the user on access policy. The server reads the file at boot.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — status and guidance

Read both state files and give the user a complete picture:

1. **Credentials** — check `~/.claude/channels/feishu/.env` for
   `FEISHU_APP_ID` and `FEISHU_APP_SECRET`. Show set/not-set; if set, show
   App ID in full and secret masked (`xxxx...xxxx`, first 4 + last 4).

2. **Domain** — check for `FEISHU_DOMAIN`. Default is `https://open.feishu.cn`.
   Show if overridden to Lark (`https://open.larksuite.com`).

3. **Access** — read `~/.claude/channels/feishu/access.json` (missing file
   = defaults: `dmPolicy: "pairing"`, empty allowlist). Show:
   - DM policy and what it means in one line
   - Allowed senders: count, and list open_ids
   - Pending pairings: count, with codes and sender IDs if any

4. **What next** — end with a concrete next step based on state:
   - No credentials → guide through Feishu setup (see Setup Guide below)
   - Credentials set, nobody allowed → *"DM your bot on Feishu. It replies
     with a code; approve with `/feishu:access pair <code>`."*
   - Credentials set, someone allowed → *"Ready. DM your bot to reach the
     assistant."*

**Push toward lockdown — always.** Once the IDs are in, pairing has done
its job. Drive the user to `/feishu:access policy allowlist`.

### `<app_id> <app_secret>` — save credentials

1. Parse `$ARGUMENTS` as two tokens: App ID and App Secret.
   - App ID looks like `cli_xxxxxxxxxx`
   - App Secret is a long hex string
2. `mkdir -p ~/.claude/channels/feishu`
3. Read existing `.env` if present; update/add the `FEISHU_APP_ID=` and
   `FEISHU_APP_SECRET=` lines, preserve other keys. Write back, no quotes.
4. `chmod 600 ~/.claude/channels/feishu/.env`
5. Confirm, then show the no-args status so the user sees where they stand.
6. Note: changes need a session restart or `/reload-plugins`.

### `domain <url>` — set domain

Set `FEISHU_DOMAIN` in `.env`. Valid values:
- `https://open.feishu.cn` (Feishu, default)
- `https://open.larksuite.com` (Lark international)

### `clear` — remove credentials

Delete the `FEISHU_APP_ID=` and `FEISHU_APP_SECRET=` lines.

---

## Setup Guide

When the user needs to create a Feishu app from scratch, walk them through:

1. Go to [Feishu Open Platform](https://open.feishu.cn/app) (or
   [Lark](https://open.larksuite.com/app) for international)
2. Create a self-built app (自建应用) → get **App ID** and **App Secret**
3. **Permissions** — add these scopes in Permission Management (权限管理):
   - `im:message:send_as_bot` — send messages
   - `im:message.p2p_msg:readonly` — read P2P messages
   - `im:message.group_at_msg:readonly` — read group @mentions
   - `im:message:update` — edit messages (for edit_message tool)
   - `im:message.reactions:read` — read reactions
   - `im:message.reactions:write_only` — add reactions (for react tool)
   - `im:resource` — download resources (for download_attachment tool)
4. **Bot capability** — in "Add Features" (添加应用能力), enable Bot (机器人)
5. **Publish** — create version in Version Management (版本管理与发布),
   submit for review, approve in admin console
6. **Events** — after the bot is published and the bridge is running:
   - In Events & Callbacks (事件与回调), select **Long Connection** (长连接)
   - Add `im.message.receive_v1` event
   - Create and publish a new version (requires admin approval)

The two-phase approach is required because Feishu validates the WebSocket
connection when saving event configuration — the bridge must be running.

---

## Implementation notes

- The channels dir might not exist if the server hasn't run yet. Missing file
  = not configured, not an error.
- The server reads `.env` once at boot. Credential changes need a session
  restart or `/reload-plugins`. Say so after saving.
- `access.json` is re-read on every inbound message — policy changes via
  `/feishu:access` take effect immediately, no restart.
