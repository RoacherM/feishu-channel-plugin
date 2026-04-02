# Feishu/Lark Channel Plugin for Claude Code

A messaging bridge that connects [Feishu](https://www.feishu.cn/) (or [Lark](https://www.larksuite.com/)) to Claude Code, with built-in access control.

## Prerequisites

1. A Feishu self-built app with:
   - **Events & Callbacks** → Subscription mode set to **Persistent Connection** (长连接)
   - **Event subscribed**: `im.message.receive_v1`
   - Permissions: "读取用户发给机器人的单聊消息", "接收群聊中@机器人消息事件"
2. [Bun](https://bun.sh/) runtime installed
3. Claude Code CLI

## Installation

```bash
# 1. Add the marketplace
/plugin marketplace add RoacherM/feishu-channel-plugin

# 2. Install the plugin
/plugin install feishu@feishu-plugin
```

## Configuration

In Claude Code, run:

```
/feishu:configure <APP_ID> <APP_SECRET>
```

This saves credentials to `~/.claude/channels/feishu/.env`.

## Usage

Start Claude Code with the Feishu channel:

```bash
claude --dangerously-load-development-channels plugin:feishu@feishu-plugin
```

Then DM your bot on Feishu — messages will be forwarded to the Claude Code session.

## Access Control

Manage who can reach your bot:

```
/feishu:access                    # Show current policy
/feishu:access pair <code>        # Approve a pending pairing
/feishu:access allow <open_id>    # Add user to allowlist
/feishu:access remove <open_id>   # Remove user
/feishu:access policy allowlist   # Lock to allowlist only (no new pairings)
/feishu:access policy pairing     # Allow new pairings (default)
```

## Features

- **Private chat (P2P)**: Direct messages to the bot are forwarded to Claude
- **Group chat**: Add the bot to a group; it responds when @mentioned
- **Access control**: Pairing flow for new users, allowlist for production
- **Image support**: Photos sent to the bot are downloaded and forwarded
- **File attachments**: Documents and other files are handled
- **Permission relay**: Tool permission prompts forwarded to Feishu for approval
- **Emoji reactions**: Acknowledgement reactions on received messages
- **Stale process cleanup**: Automatically kills zombie instances on startup to prevent WebSocket event stealing
- **Delivery queue**: Per-chat serialized delivery with dedup and retry

## Updating

```
/feishu:upgrade           # Pull latest from GitHub + clear cache
/feishu:upgrade status    # Check if updates are available
```

After upgrading, run `/reload-plugins`, then **exit and re-enter Claude Code** for the MCP server to restart with the new code.

> **Note**: Claude Code's built-in `/plugin update` has a [known bug](https://github.com/anthropics/claude-code/issues/37252) that doesn't fetch the remote before comparing versions. Use `/feishu:upgrade` instead.

## Project Structure

```
.claude-plugin/
  marketplace.json        # Marketplace definition
feishu/                   # Plugin source
  .claude-plugin/
    plugin.json           # Plugin metadata
  .mcp.json               # MCP server config
  server.ts               # Main server
  skills/
    access/SKILL.md       # /feishu:access skill
    configure/SKILL.md    # /feishu:configure skill
    upgrade/SKILL.md      # /feishu:upgrade skill
```

## License

MIT
