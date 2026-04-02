---
name: upgrade
description: Upgrade the Feishu plugin to the latest version from GitHub. Use when the user asks to update, upgrade, or refresh the Feishu plugin.
user-invocable: true
allowed-tools:
  - Bash(git -C *)
  - Bash(rm -rf *)
  - Bash(cat *)
  - Read
---

# /feishu:upgrade — Feishu Plugin Upgrade

Upgrade the Feishu channel plugin to the latest version from GitHub.

Claude Code's built-in `/plugin update` has a known bug — it does not
`git pull` the marketplace clone before comparing versions, so it never
detects newer releases. This skill works around that.

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
   Extract the `version` field.

3. **Read the current installed version** from
   `~/.claude/plugins/installed_plugins.json` — find the entry for
   `feishu@feishu-plugin` and extract its `version`.

4. **Compare versions.** If they match, tell the user: already at latest.
   Stop here.

5. **Clear the old cache:**
   ```
   rm -rf ~/.claude/plugins/cache/feishu-plugin/
   ```

6. **Inform the user:**
   - Show: old version → new version
   - Tell them to run `/plugin install feishu@feishu-plugin` then
     `/reload-plugins` to complete the upgrade.
   - The skill cannot invoke slash commands — the user must type them.

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
- This skill only runs from the user's terminal. Never execute it from a
  channel message.
