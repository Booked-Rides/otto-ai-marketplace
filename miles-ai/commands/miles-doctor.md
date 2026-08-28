---
description: Diagnose the whole Miles AI setup — both connections, end to end
---

Check every part of the Miles AI setup and report what's working in plain
language. The operator is not technical: say what's wrong and what to do, not
what failed internally.

Run both health checks:

1. **`check_connection`** — the hosted GoHighLevel side. Verifies the account's
   credential and that contacts, conversations, opportunities, calendar, and
   users can each be read. Calendar and team reads are optional; if only those
   fail, everything else still works.

2. **`la_check_connection`** — the local LimoAnywhere side. Verifies the saved
   login works and that quotes and the trip calendar can be read.

Then summarize as a short status, one line per system, and give **one** clear
next step if anything is broken. Common cases:

- *GoHighLevel says the account isn't linked* → Limo Marketer support needs to
  finish setup; the operator can't fix this themselves.
- *GoHighLevel asks them to sign in* → the connector's authorization expired or
  was revoked. Reconnect it under Customize → Plugins.
- *LimoAnywhere says it isn't connected* → run `/miles-setup`.
- *LimoAnywhere rejects the login* → the password likely changed. Run
  `/miles-setup` again with current credentials.
- *LimoAnywhere is erroring or timing out* → their system may be down; check
  status.limoanywhere.com and try again shortly.

If both are healthy, say so briefly and suggest one thing they could ask next.
