---
name: upgrade
description: Upgrade the Feishu plugin to the latest version from GitHub. Use when the user asks to update, upgrade, or refresh the Feishu plugin.
user-invocable: true
allowed-tools:
  - Bash(git -C *)
  - Bash(rm -rf *)
  - Bash(cp -r *)
  - Bash(cat *)
  - Bash(python3 -c *)
  - Read
  - Write
---

# /feishu:upgrade — Feishu Plugin Upgrade

Upgrade the Feishu channel plugin to the latest version from GitHub.

Claude Code's built-in `/plugin update` has a known bug — it does not
`git pull` the marketplace clone before comparing versions, so it never
detects newer releases. This skill works around that by pulling, copying
the new files into cache, and patching `installed_plugins.json` directly.

Arguments passed: `$ARGUMENTS`

---

## Dispatch on arguments

### No args — upgrade to latest

1. **Pull latest from remote:**
   ```
   git -C ~/.claude/plugins/marketplaces/feishu-plugin pull origin main
   ```
   If this fails (network error, merge conflict), stop and report.

2. **Read the new version** from the marketplace clone:
   ```
   cat ~/.claude/plugins/marketplaces/feishu-plugin/feishu/.claude-plugin/plugin.json
   ```
   Extract the `version` field. Also get the git commit SHA:
   ```
   git -C ~/.claude/plugins/marketplaces/feishu-plugin rev-parse HEAD
   ```

3. **Read the current installed version** from
   `~/.claude/plugins/installed_plugins.json` — find the entry for
   `feishu@feishu-plugin` and extract its `version`.

4. **Compare versions.** If they match, tell the user: already at latest.
   Stop here.

5. **Clear the old cache and copy new files:**
   ```
   rm -rf ~/.claude/plugins/cache/feishu-plugin/
   cp -r ~/.claude/plugins/marketplaces/feishu-plugin/feishu ~/.claude/plugins/cache/feishu-plugin/feishu/<NEW_VERSION>/
   ```

6. **Update installed_plugins.json** — Read the file, update the
   `feishu@feishu-plugin` entry with the new version, installPath, SHA,
   and lastUpdated timestamp. Write the file back (pretty-printed, 2-space
   indent). Use python3 for the JSON manipulation:
   ```
   python3 -c "
   import json, sys
   from datetime import datetime, timezone
   path = sys.argv[1]
   version = sys.argv[2]
   sha = sys.argv[3]
   with open(path) as f: data = json.load(f)
   entry = data['plugins']['feishu@feishu-plugin'][0]
   entry['version'] = version
   entry['installPath'] = f'{sys.argv[4]}'
   entry['gitCommitSha'] = sha
   entry['lastUpdated'] = datetime.now(timezone.utc).strftime('%Y-%m-%dT%H:%M:%S.000Z')
   with open(path, 'w') as f: json.dump(data, f, indent=2)
   print('updated')
   " ~/.claude/plugins/installed_plugins.json <NEW_VERSION> <SHA> ~/.claude/plugins/cache/feishu-plugin/feishu/<NEW_VERSION>
   ```

7. **Inform the user:**
   - Show: old version → new version
   - Tell them to run `/reload-plugins` to apply, then **exit and
     re-enter Claude Code** for the MCP server to restart with the new
     code.

### `status` — check without upgrading

1. Fetch remote refs without merging:
   ```
   git -C ~/.claude/plugins/marketplaces/feishu-plugin fetch origin
   ```

2. Compare local HEAD vs `origin/main`:
   ```
   git -C ~/.claude/plugins/marketplaces/feishu-plugin log HEAD..origin/main --oneline
   ```

3. If commits exist, show what's new and the version in
   `origin/main:.claude-plugin/marketplace.json`. Otherwise report
   "up to date."

---

## Implementation notes

- The marketplace clone lives at `~/.claude/plugins/marketplaces/feishu-plugin/`.
  This path is stable across installs — it comes from `known_marketplaces.json`.
- Only delete the cache for `feishu-plugin`, not the entire cache directory.
- Always update `installed_plugins.json` — without this, the next upgrade
  will detect the same version difference again (infinite upgrade loop).
- This skill only runs from the user's terminal. Never execute it from a
  channel message.
