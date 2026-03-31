# Feishu/Lark — Access & Delivery

A Feishu bot is addressable by anyone in the tenant. Without a gate, those messages would flow straight into your assistant session. The access model described here decides who gets through.

By default, a DM from an unknown sender triggers **pairing**: the bot replies with a 6-character code and drops the message. You run `/feishu:access pair <code>` from your assistant session to approve them. Once approved, their messages pass through.

All state lives in `~/.claude/channels/feishu/access.json`. The `/feishu:access` skill commands edit this file; the server re-reads it on every inbound message, so changes take effect without a restart. Set `FEISHU_ACCESS_MODE=static` to pin config to what was on disk at boot (pairing is unavailable in static mode since it requires runtime writes).

## At a glance

| | |
| --- | --- |
| Default policy | `pairing` |
| Sender ID | Feishu open_id (e.g. `ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`) |
| Group key | Chat ID (e.g. `oc_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`) |
| Config file | `~/.claude/channels/feishu/access.json` |

## DM policies

`dmPolicy` controls how DMs from senders not on the allowlist are handled.

| Policy | Behavior |
| --- | --- |
| `pairing` (default) | Reply with a pairing code, drop the message. Approve with `/feishu:access pair <code>`. |
| `allowlist` | Drop silently. No reply. |
| `disabled` | Drop everything, including allowlisted users and groups. |

```
/feishu:access policy allowlist
```

## User IDs

Feishu identifies users by **open_id** like `ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`. These are app-scoped and stable. The allowlist stores open_ids.

Pairing captures the open_id automatically. To find one manually, check the Feishu admin console or use the bot's API.

```
/feishu:access allow ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
/feishu:access remove ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

## Groups

Groups are off by default. Opt each one in individually.

```
/feishu:access group add oc_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Chat IDs start with `oc_`. You can find them in the Feishu admin console or via the API.

With the default `requireMention: true`, the bot responds only when @mentioned. Pass `--no-mention` to process every message, or `--allow id1,id2` to restrict which members can trigger it.

```
/feishu:access group add oc_xxx --no-mention
/feishu:access group add oc_xxx --allow ou_aaa,ou_bbb
/feishu:access group rm oc_xxx
```

## Mention detection

In groups with `requireMention: true`, any of the following triggers the bot:

- An @mention of the bot (detected via the mentions array in the event)
- A match against any regex in `mentionPatterns`

```
/feishu:access set mentionPatterns '["^hey claude\\b", "\\bassistant\\b"]'
```

## Delivery

Configure outbound behavior with `/feishu:access set <key> <value>`.

**`ackReaction`** reacts to inbound messages on receipt. Use Feishu emoji type strings (THUMBSUP, EYES, DONE, etc.) or Unicode emoji (auto-mapped).

```
/feishu:access set ackReaction EYES
/feishu:access set ackReaction ""
```

**`replyToMode`** controls threading on chunked replies. `first` (default) threads only the first chunk; `all` threads every chunk; `off` sends all chunks standalone.

**`textChunkLimit`** sets the split threshold. Default 4000.

**`chunkMode`** chooses the split strategy: `length` cuts exactly at the limit; `newline` prefers paragraph boundaries.

## Skill reference

| Command | Effect |
| --- | --- |
| `/feishu:access` | Print current state: policy, allowlist, pending pairings, enabled groups. |
| `/feishu:access pair a4f91c` | Approve pairing code `a4f91c`. |
| `/feishu:access deny a4f91c` | Discard a pending code. |
| `/feishu:access allow ou_xxx` | Add an open_id directly. |
| `/feishu:access remove ou_xxx` | Remove from the allowlist. |
| `/feishu:access policy allowlist` | Set `dmPolicy`. Values: `pairing`, `allowlist`, `disabled`. |
| `/feishu:access group add oc_xxx` | Enable a group. Flags: `--no-mention`, `--allow id1,id2`. |
| `/feishu:access group rm oc_xxx` | Disable a group. |
| `/feishu:access set ackReaction EYES` | Set a config key. |

## Config file

`~/.claude/channels/feishu/access.json`. Absent file is equivalent to `pairing` policy with empty lists.

```jsonc
{
  "dmPolicy": "pairing",
  "allowFrom": ["ou_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"],
  "groups": {
    "oc_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx": {
      "requireMention": true,
      "allowFrom": []
    }
  },
  "mentionPatterns": ["^hey claude\\b"],
  "ackReaction": "EYES",
  "replyToMode": "first",
  "textChunkLimit": 4000,
  "chunkMode": "newline"
}
```
