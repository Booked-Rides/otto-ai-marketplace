---
description: Connect Otto AI to LimoAnywhere (login stays on this machine)
---

Set up this operator's LimoAnywhere connection.

The LimoAnywhere tools run locally on this machine. Setup happens on a
one-time browser page served by the plugin itself, so the password is typed
into a normal form on this machine — never into this chat, and never sent to
Limo Marketer.

Do this:

0. **Preflight — check the machine first.** Run this in the shell:

   ```bash
   node --version; git --version
   ```

   - **If `node` is missing**, stop here — the LimoAnywhere connector runs on
     Node.js and cannot start without it, which is also why the `la_*` tools
     may be absent from this session. Tell the operator, in plain language, to
     install Node.js LTS from https://nodejs.org (the standard installer for
     their system; on a Mac, `brew install node` also works if they use
     Homebrew), then **fully quit and reopen Claude Desktop** and run
     `/otto-setup` again. Don't attempt the remaining steps until node is
     present.
   - **If `git` is missing**, the plugin can still run, but it won't receive
     updates from the marketplace. Recommend installing it — on a Mac, running
     `xcode-select --install` in Terminal; on Windows, the installer from
     https://git-scm.com — but continue with setup either way.
   - If both are present, say nothing about the preflight and move on.
   - **Windows caveat:** on Windows this shell is a Linux sandbox, so the
     check above reflects the sandbox, not the operator's machine. If the
     `la_*` tools are missing from this session on Windows, walk the operator
     through this, one step at a time, in plain language:

     1. "Do you remember installing Node.js on this computer?" If not (or
        unsure): go to **nodejs.org**, download it, click **Next** through
        every step of the installer without changing anything, **restart the
        computer**, and open Claude Desktop again. The standard installer
        sets everything up; the restart is what makes it take effect.
     2. If Node is already installed and it still doesn't work: have them
        open **PowerShell** (press the Windows key, type "powershell", press
        Enter) and paste this one line, exactly as written, then press Enter:

        ```powershell
        [Environment]::SetEnvironmentVariable("Path", [Environment]::GetEnvironmentVariable("Path","User") + ";C:\Program Files\nodejs", "User")
        ```

        Then right-click the Claude icon in the system tray (bottom-right,
        near the clock), choose **Exit**, and reopen Claude Desktop.
     3. Only if both fail, collect diagnostics for support: the
        `Using MCP server command: node with path:` line from
        `%LOCALAPPDATA%\Claude\logs\main.log`.

1. Recommend the operator create a **dedicated view-only "Otto AI" user** in
   LimoAnywhere rather than sharing their admin login. It limits what this
   connection can reach and it can be revoked on its own. If they'd rather use
   an existing login, that's their call; don't push twice.

2. Call the **`la_connect_start`** tool. Give the operator the link it returns
   and tell them to open it in a browser **on this machine** and enter their
   company ID, username, and password — the same three fields as the
   manage.mylimobiz.com login form. The page verifies the login with
   LimoAnywhere and saves it (readable only by their user) on this machine.
   The link expires after 10 minutes or one successful connection; if it
   expires, call `la_connect_start` again for a fresh one.

3. When they say they've submitted, run `la_check_connection` and report the
   result in plain language. The connection is live immediately — no restart.

4. **Fallback only** if they truly can't open the page (no browser on this
   machine): the `la_connect` tool takes the three fields as arguments. Warn
   them first that the password will appear in this conversation's history,
   and get their okay before proceeding.

If the login is rejected, the likely causes are a typo, a recently changed
password, or a user without permission for the quotes and calendar screens.

Never repeat a password back into the conversation — not in confirmations,
not in summaries.
