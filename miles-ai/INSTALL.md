# Installing the Miles AI Cowork plugin (local testing)

This guide takes a clean checkout to a working plugin inside Claude Cowork. It
assumes no prior context — if you're picking this up from someone else, start
here.

**What you're installing:** one plugin containing two halves.

- **LimoAnywhere** runs *on your machine* as a bundled MCP server, signed in as
  your own LimoAnywhere user. Strictly read-only.
- **GoHighLevel** runs *on Limo Marketer's hosted server* (Railway) and
  authenticates through Booked Rides. The analysis and the write-approval gate
  live there, not on your machine.

Both install in a single step and appear together in one session.

---

## 1. Prerequisites

| Requirement | Notes |
|---|---|
| **Node.js 18+** | `node --version`. Needed to build; the plugin bundles its own copy of the compiled server. |
| **Claude Cowork** | Any paid plan (Pro, Max, Team, Enterprise). Cowork is not available on Free. |
| **Latest Claude Desktop** | Cowork requires it. |
| **Windows only:** Virtual Machine Platform enabled | Cowork runs in a lightweight VM. Without this Windows feature it won't start. |
| **A LimoAnywhere login** | Company ID, username, password — the three fields from the manage.mylimobiz.com form. **Use a dedicated view-only "Miles AI" user**, not an admin login. |
| **A Booked Rides login** | For the hosted GoHighLevel connector. The account must have a GHL location linked, or the GHL tools will say so and stop. |
| **The Railway domain** | See step 3. Ask whoever handed this off if you don't have it. |

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
  ok — 8 LimoAnywhere tools

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

## 3. Set the hosted URL — required

Open `plugins/miles-ai/.mcp.json`. It ships with a placeholder:

```json
"url": "https://SET-YOUR-RAILWAY-DOMAIN-HERE/mcp"
```

Replace that host with the real Railway domain, then re-run
`npm run plugin:build`. The build prints a warning while the placeholder is
still in place.

**If you skip this:** the LimoAnywhere half still works. The GoHighLevel half
won't connect.

---

## 4. Install into Cowork

1. Zip the package so its contents sit at the **root** of the archive (not
   wrapped in a `miles-ai/` folder) — the upload dialog accepts `.zip` only:

   ```bash
   cd plugins/miles-ai && zip -r ../../miles-ai-plugin.zip . -x "*.DS_Store"
   ```

2. Open Claude Desktop, then **Customize** in the sidebar, then **Plugins**.
3. Choose the **upload** option (not the marketplace) and select
   `miles-ai-plugin.zip`.

> **Why upload rather than the marketplace?** `plugins/miles-ai/server/` is
> generated and gitignored, so cloning the repo alone doesn't give you a
> runnable plugin — you have to build it. A separate marketplace repo holding
> built packages is the planned distribution path; it doesn't exist yet.

After installing, open the plugin. You should see:

- **Skills:** `gohighlevel`, `limoanywhere`
- **Connectors:** `miles-limoanywhere` (local), `miles-gohighlevel` (remote)
- **Commands:** `/miles-setup`, `/miles-doctor`

If the GoHighLevel connector prompts you to sign in, that's the Booked Rides
OAuth flow — use the test account.

---

## 5. Connect LimoAnywhere

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

## 6. Verify

```
/miles-doctor
```

This runs both health checks and reports in plain language. Healthy looks like:
GoHighLevel connected and reading contacts, conversations and opportunities;
LimoAnywhere logged in and reading quotes and the calendar.

Then try a few real questions:

| Ask | Exercises |
|---|---|
| "What's on the calendar today?" | local LA server, calendar JSON endpoint |
| "How many quote requests came in last week?" | local LA server, list scraping |
| "Show me my new leads from the last 7 days" | hosted GHL connector |
| "What does the pipeline look like?" | hosted GHL connector |
| "Who should I follow up with today?" | the follow-up skill workflow |

---

## What the first install test found (macOS, Aug 2026)

The two questions this test existed to answer are now answered — on macOS.

**1. Does Cowork run the bundled Node server?** Yes — but not in a sandbox.
Claude Desktop launches it **directly on the host machine**, as the local
user, using whatever `node` is on the PATH (an nvm install, in the test).
Implication for distribution: a client without Node installed will see the
connector fail to start; bundling a runtime (per-OS) remains the fix for that.

**2. Is outbound HTTPS to `manage.mylimobiz.com` possible?** Yes — the server
runs on the host, so it logs in from the operator's own IP exactly as
designed. Verified with a live login, quote read, and calendar read.

**Also learned:** Cowork passes the connector's `env` through **without
defining `${CLAUDE_PLUGIN_DATA}`**, so `MILES_LA_CONFIG` arrives as a literal
unexpanded string. The server therefore treats any value containing `${` as
unset and falls back to `~/.claude/plugins/data/miles-ai/la-credentials.json`.
Windows and Linux (via Claude Code) remain untested — report findings from
those.

---

## Troubleshooting

| Symptom | Cause and fix |
|---|---|
| `plugin:build` says `dist/laServer.js is missing` | Run `npm run build` first, or use `npm run plugin:build`, which does both. |
| Build warns about `SET-YOUR-RAILWAY-DOMAIN-HERE` | Step 3 wasn't done. LimoAnywhere works; GoHighLevel won't. |
| Plugin installs but shows no connectors | Check `plugins/miles-ai/server/laServer.mjs` exists. If not, the build didn't finish. |
| Upload rejected: "path with invalid characters" | The zip contains `node_modules` (or other `@`-prefixed paths) from an old build. Re-run `npm run plugin:build` and re-zip. |
| `miles-limoanywhere` fails to start | Sandbox runtime problem — that's finding #1 above. Report it. |
| LA tools say "isn't connected yet" | Run `/miles-setup` — its `la_connect` call verifies and saves the login, live immediately. If it claims success but tools still say this, check the file exists at `~/.claude/plugins/data/miles-ai/la-credentials.json` — the server's default. (Cowork does not expand `${CLAUDE_PLUGIN_DATA}` in the connector's `MILES_LA_CONFIG`, so the env var is ignored unless it's a real path.) |
| LA says the login was rejected | Wrong credentials, a changed password, or a user without permission for the quotes and calendar screens. |
| GHL says the account isn't linked | The Booked Rides account has no GHL location linked. Not fixable locally — needs Limo Marketer support. |
| GHL asks you to sign in repeatedly | The connector's OAuth grant expired or was revoked. Reconnect under Customize → Plugins. |

---

## Repo orientation

If you're working on this rather than only testing it:

- `src/laServer.ts` — the local LimoAnywhere MCP server entrypoint. Registers
  only `LA_TOOLS`, and deliberately has no GoHighLevel credential, vault, or
  write gate available to it.
- `src/la/` — the LimoAnywhere access layer. **Read-only by construction:**
  every request funnels through `laFetch`, whose `LA_READ_ALLOWLIST` refuses
  anything not on the known read screens. `npm run test:isolation` audits this.
- `src/tools/limoanywhere/` — the eight `la_*` tools.
- `plugins/miles-ai/` — the plugin package (skills, commands, manifests).
- `scripts/build-plugin.mjs` — assembles and verifies the package.
- `.claude/plan/distribution-model-review.md` — why the surfaces split this way.

Read `CLAUDE.md` at the repo root before changing anything. It documents the
invariants the test gates enforce: LimoAnywhere is strictly read-only, every
GoHighLevel write goes through the prepare/confirm approval gate, and no tool
ever accepts a location ID as input.
