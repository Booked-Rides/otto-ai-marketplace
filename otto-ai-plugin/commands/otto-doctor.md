---
description: Diagnose the Otto AI setup and report in plain language
---

Check the Otto AI setup and report what's working in plain language. The
operator is not technical: say what's wrong and what to do, not what failed
internally.

1. Check the machine itself. Run in the shell:

   ```bash
   node --version; git --version; uname -s
   ```

   Node.js is what the local LimoAnywhere connector runs on. If anything is
   missing, **fix it for the operator** instead of pointing them at a
   website — the full playbook is `/otto-setup`'s preflight (step 0); follow
   it from here. In short: on a Mac, install Node in this chat (`brew install
   node` if Homebrew exists, else the nvm two-liner) and have them fully quit
   and reopen Claude Desktop; on Windows, give them the single `winget`
   PowerShell paste that installs Node and Git together, then a computer
   restart. Git missing on a Mac: `brew install git`, or run
   `xcode-select --install` and tell them to click Install in the popup —
   updates won't arrive without it, but today's session still works. If both
   are present, just include "machine setup: OK" in the report.

   **If the `la_*` tools are absent from this session entirely**, check the
   causes in `/otto-setup` step 0-A **in order**: first that Cowork is running
   this task locally (Settings → Cowork → "Run new tasks in the cloud" must
   be OFF; org admins control it on Team/Enterprise — the connector can never
   appear in a cloud task), then missing Node.

   **Windows:** this shell is a Linux sandbox, so the check reflects the
   sandbox, not the operator's machine — a passing check does NOT prove the
   host can spawn the connector. If the `la_*` tools are absent on Windows,
   don't diagnose PATHs at the operator; use `/otto-setup`'s escalation path
   (winget paste + restart; then the PATH one-liner), and only collect
   `%LOCALAPPDATA%\Claude\logs\main.log` diagnostics if those fail.

2. Run **`la_check_connection`** — the local LimoAnywhere side. It verifies
   the saved login works and that quotes and the trip calendar can be read.

3. If the hosted Otto AI GoHighLevel connector is also present in this
   session (a `check_connection` tool exists), run it too and include it in
   the report. If it isn't present, don't treat that as a problem — it's a
   separate connector, not part of this plugin.

Then summarize as a short status, one line per system, and give **one** clear
next step if anything is broken. Common cases:

- *The `la_*` tools aren't in this session at all* → either Cowork ran this
  task in the cloud (fix the "Run new tasks in the cloud" setting, restart,
  new task) or Node.js is missing on the machine (install it for them per
  `/otto-setup` step 0, then fully quit and reopen Claude Desktop).
- *LimoAnywhere says it isn't connected* → run `/otto-setup`.
- *Something that was supposedly fixed is still broken* → the plugin may be
  stale; auto-update is off by default for this marketplace. Run
  `/otto-update`.
- *LimoAnywhere rejects the login* → the password likely changed. Run
  `/otto-setup` again with current credentials.
- *LimoAnywhere is erroring or timing out* → their system may be down; check
  status.limoanywhere.com and try again shortly.
- *GoHighLevel (if connected) says the login isn't linked* → Limo Marketer
  support needs to finish setup; the operator can't fix this themselves.
- *GoHighLevel (if connected) asks them to sign in* → the connector's
  authorization expired. Reconnect it in Claude's connector settings.

If everything is healthy, say so briefly and suggest one thing they could ask
next.
