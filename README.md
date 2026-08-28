# Otto AI Marketplace

Limo Marketer's plugin marketplace for Claude. It currently ships one plugin:

**Miles AI** — ask questions about your LimoAnywhere back office: today's
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

3. Install **Miles AI**.
4. In a session, run `/miles-setup` to connect LimoAnywhere — Claude gives you
   a link to a page on your own computer where you enter your LimoAnywhere
   login. It's saved only on your machine.

Then try: *"What's on the calendar today?"* or *"How much is booked for next
month?"* Run `/miles-doctor` any time to check the connection.

## The GoHighLevel side (optional, separate)

Leads, conversations, and pipeline live in a separate hosted connector — not
in this plugin. Add it in Claude's connector settings by URL:

```
https://miles-ai-production.up.railway.app/mcp
```

It signs in with your Booked Rides account. With both connected, Miles can
answer across marketing and operations together.

## Updates

When a new version is published here, Claude shows an update for Miles AI
under **Customize → Plugins** — one click, and your saved login is untouched.

## For Limo Marketer engineers

This repo holds the **built** package — clients can't run a build. Don't edit
`miles-ai/` here by hand: it's generated from the
[miles-ai](https://github.com/Booked-Rides/miles-ai) repo by
`npm run plugin:release`, which rebuilds, verifies the packaged server boots,
refuses to reuse a version number, and stages the copy here for you to review
and push.
