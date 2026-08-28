# Installing the Miles AI Cowork plugin

This guide takes a clean checkout to a working plugin inside Claude Cowork. It
assumes no prior context — if you're picking this up from someone else, start
here.

**What the plugin is:** the **LimoAnywhere** half of Miles AI. It runs *on the
operator's machine* as a bundled MCP server, signed in as their own
LimoAnywhere user, strictly read-only — trip calendar, quotes, reservations,
booked revenue.

**What the plugin is not:** GoHighLevel. That half stays a **separate hosted
MCP connector** on Limo Marketer's server (Railway), added by URL
(`https://miles-ai-production.up.railway.app/mcp`) and authenticated through
Booked Rides. The analysis and the write-approval gate live there. Operators
can use the plugin with or without it.

---

## 1. Prerequisites

| Requirement | Notes |
|---|---|
| **Node.js 18+** | `node --version`. Needed to build, and the installed plugin currently uses the machine's `node` (Cowork launches the server on the host with the PATH's node). |
| **Claude Cowork** | Any paid plan (Pro, Max, Team, Enterprise). Cowork is not available on Free. |
| **Latest Claude Desktop** | Cowork requires it. |
| **Windows only:** Virtual Machine Platform enabled | Cowork's requirement. |
| **A LimoAnywhere login** | Company ID, username, password — the three fields from the manage.mylimobiz.com form. **Use a dedicated view-only "Miles AI" user**, not an admin login. |

---

## 2. Build the package

```bash
npm install
npm run plugin:build
```

`plugin:build` compiles the TypeScript, bundles the LimoAnywhere server and
its runtime dependencies into a single file
(`plugins/miles-ai/server/laServer.mjs`), then **starts the packaged server
to verify it runs standalone**. (A single bundled file rather than a vendored
`node_modules`: Cowork's upload validator rejects zip paths containing `@`,
which npm's scoped package directories use.) You should see:

```
Verifying the packaged server starts standalone...
  ok — 10 LimoAnywhere tools

Plugin package ready: plugins/miles-ai
```

If that verification fails, stop — the package won't work in Cowork either.

Optionally confirm the checkout is healthy first. All five gates pass with no
`.env` and no network access:

```bash
npm run smoke          # stdio server boots, registers tools, degrades gracefully
npm run smoke:la       # the local LimoAnywhere server specifically
npm run smoke:http     # full hosted OAuth flow against an in-memory mock
npm run test:isolation # two-tenant isolation + read-only audit
npm run test:la-creds  # LimoAnywhere credential linking
```

---

## 3. Install into Cowork

**Clients install from the marketplace** — one time, updates flow
automatically:

1. Claude Desktop → **Customize → Plugins → Add marketplace** →
   `Booked-Rides/otto-ai-marketplace`.
2. Install **Miles AI**.

**For local testing of an unreleased build**, upload a zip instead. The
contents must sit at the **root** of the archive (not wrapped in a
`miles-ai/` folder) — the upload dialog accepts `.zip` only:

```bash
cd plugins/miles-ai && zip -r ../../miles-ai-plugin.zip . -x "*.DS_Store"
```

After installing, open the plugin. You should see:

- **Skill:** `limoanywhere`
- **Connector:** `miles-limoanywhere` (local)
- **Commands:** `/miles-setup`, `/miles-doctor`

---

## 4. Connect LimoAnywhere

Run:

```
/miles-setup
```

Claude calls the plugin's `la_connect_start` tool and gives you a one-time
link like `http://127.0.0.1:PORT/setup?token=…`. Open it in a browser on this
machine and enter your company ID, username, and password — a normal login
form, served by the plugin itself. It verifies the login with LimoAnywhere
and — only if it's accepted — saves it to a file on your machine, readable
only by your user. It takes effect immediately: no restart, no new session.

The password never enters the chat and is never sent to Limo Marketer — it
goes browser → local plugin process → file, all on this machine. The link
expires after 10 minutes or one successful connection. (A fallback tool,
`la_connect`, accepts the login as chat input for machines with no browser;
Claude will warn you before using it.)

---

## 5. Verify

```
/miles-doctor
```

Healthy looks like: LimoAnywhere logged in and reading quotes and the
calendar. Then try:

| Ask | Exercises |
|---|---|
| "What's on the calendar today?" | calendar JSON endpoint |
| "How many quote requests came in last week?" | list scraping |
| "How much is booked for next month?" | revenue summary |
| "Which quotes didn't convert?" | quote conversion report |

## 6. Optional: the GoHighLevel connector

The marketing half (leads, conversations, pipeline, the approval-gated
writes) is a separate remote connector, not part of this plugin. Add it in
Claude's connector settings by URL:

```
https://miles-ai-production.up.railway.app/mcp
```

It authenticates through the operator's Booked Rides login. `/miles-doctor`
will include it in its report whenever it's present.

---

## What the first install test found (macOS, Aug 2026)

- **Cowork runs the bundled Node server, but not in a sandbox.** Claude
  Desktop launches it **directly on the host machine**, as the local user,
  using whatever `node` is on the PATH (an nvm install, in the test).
  Implication: a client without Node installed will see the connector fail to
  start; bundling a runtime (per-OS) remains the fix for that.
- **Outbound HTTPS to `manage.mylimobiz.com` works** — the server runs on the
  host, so it logs in from the operator's own IP exactly as designed.
- **Cowork passes the connector's `env` through without defining
  `${CLAUDE_PLUGIN_DATA}`**, so `MILES_LA_CONFIG` arrives as a literal
  unexpanded string. The server treats any value containing `${` as unset and
  falls back to `~/.claude/plugins/data/miles-ai/la-credentials.json`.
- Windows and Linux (via Claude Code) remain untested — report findings from
  those.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `plugin:build` says `dist/laServer.js is missing` | Run `npm run build` first, or use `npm run plugin:build`, which does both. |
| Plugin installs but shows no connector | Check `plugins/miles-ai/server/laServer.mjs` exists. If not, the build didn't finish. |
| Upload rejected: "path with invalid characters" | The zip contains `node_modules` (or other `@`-prefixed paths) from an old build. Re-run `npm run plugin:build` and re-zip. |
| `miles-limoanywhere` fails to start | Most likely no `node` on the machine's PATH. |
| LA tools say "isn't connected yet" | Run `/miles-setup` — its `la_connect_start` page verifies and saves the login, live immediately. If it claims success but tools still say this, check the file exists at `~/.claude/plugins/data/miles-ai/la-credentials.json` — the server's default. (Cowork does not expand `${CLAUDE_PLUGIN_DATA}` in the connector's `MILES_LA_CONFIG`, so the env var is ignored unless it's a real path.) |
| LA says the login was rejected | Wrong credentials, a changed password, or a user without permission for the quotes and calendar screens. |

---

## Repo orientation

If you're working on this rather than only testing it:

- `src/laServer.ts` — the local LimoAnywhere MCP server entrypoint. Registers
  only `LA_TOOLS` + `LA_SETUP_TOOLS`, and deliberately has no GoHighLevel
  credential, vault, or write gate available to it.
- `src/la/` — the LimoAnywhere access layer. **Read-only by construction:**
  every request funnels through `laFetch`, whose `LA_READ_ALLOWLIST` refuses
  anything not on the known read screens. `npm run test:isolation` audits this.
- `src/tools/limoanywhere/` — the eight `la_*` read tools plus the two setup
  tools (`la_connect_start`, `la_connect`).
- `plugins/miles-ai/` — the plugin package (skill, commands, manifests).
- `scripts/build-plugin.mjs` — assembles and verifies the package.
- `scripts/release-plugin.mjs` — stages a release into the marketplace repo.
- `.claude/plan/distribution-model-review.md` — why the surfaces split this way.

Read `CLAUDE.md` at the repo root before changing anything. It documents the
invariants the test gates enforce: LimoAnywhere is strictly read-only, every
GoHighLevel write goes through the prepare/confirm approval gate, and no tool
ever accepts a location ID as input.
