# Otto AI Marketplace

Limo Marketer's plugin marketplace for Claude. It currently ships one plugin:

**Otto AI by Limo Marketer** — ask questions about your LimoAnywhere back office: today's
schedule, quote requests, reservations, and booked revenue. It runs on your
own computer, signed in as your own LimoAnywhere user, and is strictly
read-only.

## Install (once)

In **Claude Desktop** (any paid plan):

1. Open **Customize → Plugins**.
2. Choose **Add marketplace** and enter:

   ```
   Booked-Rides/otto-ai-marketplace
   ```

3. Install **Otto AI**.
4. In a session, run `/otto-setup` to connect LimoAnywhere — Claude gives you
   a link to a page on your own computer where you enter your LimoAnywhere
   login. It's saved only on your machine.

Then try: *"What's on the calendar today?"* or *"How much is booked for next
month?"* Run `/otto-doctor` any time to check the connection.

## Updates

When a new version is published here, Claude shows an update for Otto AI
under **Customize → Plugins** — one click, and your saved login is untouched.
