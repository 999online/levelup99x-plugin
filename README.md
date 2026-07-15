# LevelUp99x — Jin Coach plugin

**Jin, in your terminal.** This Claude Code plugin brings LevelUp99x's Pro-tier AI companion out
of the System window and into the place you actually train. Run `/jin` and Jin fetches your live
quest line from the LevelUp99x API, coaches you through the current quest — inspecting your real
repo for coding topics, drilling recall for everything else — runs the Knowledge Check
conversationally, and pushes the completion back through the real progression pipeline: XP,
levels, ranks (E→S), your daily streak, and trophies, all reflected in the app instantly.

The plugin itself is generic and free — no content, no secrets baked in. The product is the
authenticated sync loop, gated server-side by your subscription tier. Coaching inference runs on
your own Claude subscription.

- **Pro learners**: Jin coaches you end to end and syncs every completion.
- **Free learners**: Jin is sleeping. The plugin will tell you how to wake him. 💤

## Install

Two commands inside Claude Code:

```
/plugin marketplace add 999online/levelup99x-plugin
/plugin install levelup-coach@levelup99x-plugin
```

No `npm install`, no dependencies — the bundled MCP server is a single zero-dependency Node 18+
script.

## Connect

1. In the LevelUp99x app, open **Settings → API token** and mint your personal token (starts with
   `blu_` — it's shown once).
2. In Claude Code, run:

   ```
   /jin connect blu_...your-token...
   ```

   The token is saved to `~/.levelup99x.json` (file mode `0600`) and takes effect immediately —
   no restart. You can also just run `/jin` and Jin will walk you through it.
3. Train: `/jin`.

Revoking the token in Settings locks the plugin out instantly. `LEVELUP_TOKEN` (env) overrides the
config file for CI/dev; the API URL can be overridden for local dev via the `apiUrl` option of the
connect flow (`http://localhost:8787`).

## What leaves your machine (privacy allowlist)

This plugin is built around an explicit allowlist — **"THE LINE"** — enforced in the bundled MCP
server's payload builders (and again server-side with strict schemas). The **only** data ever sent
to the LevelUp99x API:

| Sent | Example |
| --- | --- |
| Quest / skill IDs | `q_abc123` |
| Your chosen Knowledge Check answer indexes | `[2, 0, 1, 3]` |
| Completion intents | an empty `{}` POST to the complete endpoint |
| Plugin version | `levelup-coach/0.1.0` header |

**Never sent — by construction:**

- file contents or code, from your repo or anywhere else
- file or directory paths
- repository, project, or company names
- environment variable values or secrets

When Jin inspects your repo to coach a technical quest, everything he sees stays local. The MCP
server strips any field outside the allowlist before a request is built, correct check answers
never reach your machine before you submit, and your token is never logged or echoed. Verify all
of it yourself: `plugins/levelup-coach/mcp/server.js` (and run
`node plugins/levelup-coach/mcp/server.js --selftest`).

## Layout

```
.claude-plugin/marketplace.json        marketplace index (this repo)
plugins/levelup-coach/
  .claude-plugin/plugin.json           plugin manifest
  .mcp.json                            MCP server registration
  commands/jin.md                      /jin entry point
  skills/coach/SKILL.md                Jin's coaching flow (persona, domain branch, guardrails)
  mcp/server.js                        zero-dep stdio MCP server (5 levelup_* tools)
```

## Troubleshooting

- **"token invalid or revoked"** — mint a fresh token in Settings and `/jin connect` again.
- **"Jin is sleeping (Pro feature)"** — upgrade to Pro in the app; the next call picks it up, no
  reinstall.
- **Rate limited** — the message includes how long to wait.
- **Self-check** — `node plugins/levelup-coach/mcp/server.js --selftest` validates the allowlist,
  URL building, and config handling with zero network access.

Requires Node 18+. Works on macOS, Linux, and WSL.
