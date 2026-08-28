# Otto AI Marketplace

Limo Marketer's plugin marketplace for Claude. It currently ships one plugin:

**Miles AI** — ask questions about your leads, pipeline, quotes, reservations
and schedule, across GoHighLevel and LimoAnywhere.

## Install (once)

In **Claude Desktop** (any paid plan):

1. Open **Customize → Plugins**.
2. Choose **Add marketplace** and enter:

   ```
   Booked-Rides/otto-ai-marketplace
   ```

3. Install **Miles AI** from the list.
4. When the GoHighLevel connector asks you to sign in, use your Booked Rides
   login.
5. In a session, run `/miles-setup` to connect LimoAnywhere — Claude gives you
   a link to a page on your own computer where you enter your LimoAnywhere
   login. It's saved only on your machine.

Then try: *"What's on the calendar today?"* or *"Show me my new leads from
the last 7 days."* Run `/miles-doctor` any time to check both connections.

## Updates

When a new version is published here, Claude shows an update for Miles AI
under **Customize → Plugins** — one click, and your saved logins are
untouched.

## For Limo Marketer engineers

This repo holds the **built** package — clients can't run a build. Don't edit
`miles-ai/` here by hand: it's generated from the
[miles-ai](https://github.com/Booked-Rides/miles-ai) repo by
`npm run plugin:release`, which rebuilds, verifies the packaged server boots,
refuses to reuse a version number, and stages the copy here for you to review
and push.
