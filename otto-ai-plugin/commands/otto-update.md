---
description: Update Otto AI to the latest release from the marketplace
---

Update this plugin to the latest release. The operator is not technical:
report what you did and what to do next in plain language, never raw paths or
JSON unless something needs fixing.

Background you need: Claude Desktop keeps a git clone of the Otto AI
marketplace at `~/.claude/plugins/marketplaces/otto-ai` and records installs
in `~/.claude/plugins/installed_plugins.json`. Auto-update is off by default
for non-Anthropic marketplaces, so this clone goes stale until something
refreshes it — that's what this command does.

Do this:

1. **Preflight.** Run `git --version`. If git is missing, updates can't work:
   tell the operator to install it (macOS: run `xcode-select --install` in
   Terminal; Windows: the installer from https://git-scm.com), then stop.

2. **Find the marketplace clone.** Expect `~/.claude/plugins/marketplaces/otto-ai`
   with a git origin containing `otto-ai-marketplace`. If the folder name
   differs, find the right one by grepping
   `~/.claude/plugins/known_marketplaces.json` for `otto-ai-marketplace`. If
   there's no clone at all, the marketplace was never added: tell the operator
   to open **Customize → Plugins → Add marketplace**, enter
   `Booked-Rides/otto-ai-marketplace`, install **Otto AI by Limo Marketer**,
   and stop.

3. **Refresh it.** The clone is shallow, so fetch and reset rather than pull:

   ```bash
   git -C ~/.claude/plugins/marketplaces/otto-ai fetch --depth 1 origin main
   git -C ~/.claude/plugins/marketplaces/otto-ai reset --hard origin/main
   ```

   Then read the latest version from the clone's
   `.claude-plugin/marketplace.json`.

4. **Update the installed copy.** In
   `~/.claude/plugins/installed_plugins.json`, find the entry for this
   marketplace (the key ends in `@otto-ai`) and note its `installPath` and
   `version`.

   - **`otto-ai-plugin@otto-ai`, version already the latest** → say they're
     up to date and stop.
   - **`otto-ai-plugin@otto-ai`, older version** → copy the refreshed plugin
     over the installed files, in place:

     ```bash
     cp -R ~/.claude/plugins/marketplaces/otto-ai/otto-ai-plugin/. "<installPath>/"
     ```

     Don't delete anything and don't edit `installed_plugins.json` — the
     version it displays may lag behind, but the files are what run.
   - **`miles-ai@otto-ai`** → this install predates the plugin's rename and
     can't be updated in place. One-time fix, in **Customize → Plugins**:
     uninstall the old plugin, then install **Otto AI by Limo Marketer** from
     the marketplace (already refreshed in step 3). Their saved LimoAnywhere
     login survives this — the new version still reads it, no `/otto-setup`
     needed.
   - **No `@otto-ai` entry** → the plugin was installed some other way (e.g. a
     zip upload). Recommend installing from the marketplace instead so future
     updates work, via **Customize → Plugins → Add marketplace**.

5. **Finish.** If files changed, tell the operator to **fully quit and reopen
   Claude Desktop** — the update takes effect on restart — then run
   `/otto-doctor` in a new chat to confirm. Mention once that they can make
   this automatic by enabling auto-update for the **otto-ai** marketplace in
   the plugin manager (Claude Desktop leaves it off by default for
   third-party marketplaces).
