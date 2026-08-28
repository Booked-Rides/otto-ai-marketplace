---
description: Connect Otto AI to LimoAnywhere (login stays on this machine)
---

Set up this operator's LimoAnywhere connection.

The LimoAnywhere tools run locally on this machine. Setup happens on a
one-time browser page served by the plugin itself, so the password is typed
into a normal form on this machine — never into this chat, and never sent to
Limo Marketer.

Do this:

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
