---
description: Diagnose the Otto AI setup and report in plain language
---

Check the Otto AI setup and report what's working in plain language. The
operator is not technical: say what's wrong and what to do, not what failed
internally.

1. Check the machine itself. Run in the shell:

   ```bash
   node --version; git --version
   ```

   Node.js is what the local LimoAnywhere connector runs on — if it's missing,
   that alone explains any absent `la_*` tools: report that the operator needs
   to install Node.js LTS from https://nodejs.org, then fully quit and reopen
   Claude Desktop. Git missing means marketplace plugin updates won't arrive
   (macOS: `xcode-select --install`; Windows: https://git-scm.com). If both
   are present, just include "machine setup: OK" in the report.

2. Run **`la_check_connection`** — the local LimoAnywhere side. It verifies
   the saved login works and that quotes and the trip calendar can be read.

3. If the hosted Otto AI GoHighLevel connector is also present in this
   session (a `check_connection` tool exists), run it too and include it in
   the report. If it isn't present, don't treat that as a problem — it's a
   separate connector, not part of this plugin.

Then summarize as a short status, one line per system, and give **one** clear
next step if anything is broken. Common cases:

- *The `la_*` tools aren't in this session at all* → almost always Node.js
  missing (the step-1 check confirms it). Install Node.js LTS, then fully
  quit and reopen Claude Desktop.
- *LimoAnywhere says it isn't connected* → run `/otto-setup`.
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
