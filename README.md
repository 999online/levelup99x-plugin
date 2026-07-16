# LevelUp99x — Jin Coach plugin

**Jin, in your terminal.** This Claude Code plugin brings LevelUp99x's Pro-tier AI companion out
of the System window and into the place you actually train — as a **mentor, not an examiner**.
Run `/jin` and Jin fetches your live quest-line plan from the LevelUp99x API, inspects your real
repo to see how far you've actually come, syncs your true position back to the app (skip-ahead
**placement** — position credit, zero XP), coaches you through the next real step, and renders a
local HTML progress dashboard.

The split is deliberate: **the terminal is where the work happens, the app is where XP is
claimed.** For coding quests Jin never grades and never completes — when your work would honestly
pass review, he points you to the app to clear the Knowledge Check and collect the XP, streak,
and rank movement. Drill quests (language, music, history, …) keep the full conversational flow
in the terminal: teach → recall drills → Knowledge Check (clickable choices, hint-friendly) →
completion with the real celebration payload.

The plugin itself is generic and free — no content, no secrets baked in. The product is the
authenticated sync loop, gated server-side by your subscription tier. Coaching inference runs on
your own Claude subscription.

- **Pro learners**: Jin assesses, places, coaches, and syncs.
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

## What a session looks like

1. **Situate** — Jin pulls your learner state and the full quest-line map (every quest with its
   status: `locked` / `available` / `validated` / `completed`).
2. **Assess** (coding lines) — Jin reads your actual repo locally and maps what you've built
   against each quest's objectives.
3. **Place** — if your code already demonstrates quests ahead of your position, Jin proposes a
   skip: those quests become **validated** (◈) — the app moves you forward, **zero XP** — but
   only after you explicitly confirm. The final quest of a line can never be validated.
4. **Coach** — the active quest as real work in your own repo. You drive; Jin guides.
5. **Hand off** — objectives met? Claim the quest in the app: Knowledge Check, XP, streak,
   trophies all happen there.
6. **Dashboard** — Jin writes `~/.levelup99x/progress-<skill>.html` (self-contained, local-only)
   with the quest map, your placement evidence, and what's next — and opens it in your browser.

## What leaves your machine (privacy allowlist)

This plugin is built around an explicit allowlist — **"THE LINE"** — enforced in the bundled MCP
server's payload builders (and again server-side with strict schemas). The **only** data ever sent
to the LevelUp99x API:

| Sent | Example |
| --- | --- |
| Quest / skill IDs | `q_abc123`, placement target `{ "upToQuestId": 42 }` |
| Your chosen Knowledge Check answer indexes (drill quests) | `[2, 0, 1, 3]` |
| Completion intents (drill quests) | an empty `{}` POST to the complete endpoint |
| Plugin version | `levelup-coach/0.2.0` header |

**Never sent — by construction:**

- file contents or code, from your repo or anywhere else
- file or directory paths
- repository, project, or company names
- environment variable values or secrets
- placement "evidence" — Jin's assessment of your code stays in the local dashboard only

When Jin inspects your repo to place or coach you, everything he sees stays local. The MCP server
strips any field outside the allowlist before a request is built, correct check answers never
reach your machine before you submit, and your token is never logged or echoed. Verify all of it
yourself: `plugins/levelup-coach/mcp/server.js` (and run
`node plugins/levelup-coach/mcp/server.js --selftest`).

## Layout

```
.claude-plugin/marketplace.json        marketplace index (this repo)
plugins/levelup-coach/
  .claude-plugin/plugin.json           plugin manifest
  .mcp.json                            MCP server registration
  commands/jin.md                      /jin entry point
  skills/coach/SKILL.md                Jin's coaching flow (persona, assess/place/coach, guardrails)
  mcp/server.js                        zero-dep stdio MCP server (7 levelup_* tools)
```

## Troubleshooting

- **"token invalid or revoked"** — mint a fresh token in Settings and `/jin connect` again.
- **"Jin is sleeping (Pro feature)"** — upgrade to Pro in the app; the next call picks it up, no
  reinstall.
- **Rate limited** — the message includes how long to wait.
- **Placement rejected on the last quest** — by design: the final quest of a line is always
  cleared for real, never validated.
- **Self-check** — `node plugins/levelup-coach/mcp/server.js --selftest` validates the allowlist,
  URL building, and config handling with zero network access.

Requires Node 18+. Works on macOS, Linux, and WSL.
