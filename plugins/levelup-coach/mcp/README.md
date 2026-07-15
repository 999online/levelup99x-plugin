# levelup-coach MCP server

The sync path for the `levelup-coach` plugin. The `/jin` coach skill never calls the LevelUp99x
API directly — it calls these tools, and this server builds every request from an explicit
allowlist, so no file content, path, repo name, or env value can leak ("THE LINE").

**Zero dependencies.** `node server.js` works the moment the plugin is installed — no
`npm install`, no `node_modules`. Requires Node 18+ (built-in `fetch`). Optional self-check:

```bash
node server.js --selftest
```

## Tools

| Tool | Calls | Returns |
| --- | --- | --- |
| `levelup_save_config` | writes `~/.levelup99x.json` | `{ saved, path }` |
| `levelup_state` | `GET /api/coach/state` | `{ entitled, learner: { level, streak, totalXp }, focusSkill }` |
| `levelup_get_quest` | `GET /api/coach/quest[?skillId=]` | `{ skill, quest }` (checks without answers) |
| `levelup_submit_check` | `POST /api/coach/quest/:id/check` | `{ results: [{ correct, explanation }], passed, perfect }` |
| `levelup_complete_quest` | `POST /api/coach/quest/:id/complete` | `{ xpGained, perfectBonus, ..., trophies, nextQuest }` |

## Config (no env vars required)

The learner pastes their `blu_` token once when running `/jin`; the skill calls
`levelup_save_config`, which writes `~/.levelup99x.json` (mode `0600`, re-asserted with chmod on
every save). The server reads it **lazily on every call**, so a token saved mid-session works with
no restart. The token is never logged or echoed back — not in results, not in error messages.

Resolution order:

| Setting | Order |
| --- | --- |
| token | `LEVELUP_TOKEN` env (dev override) → `~/.levelup99x.json` `.token` |
| apiUrl | `LEVELUP_API_URL` env → `~/.levelup99x.json` `.apiUrl` → `DEFAULT_API_URL` constant |

`DEFAULT_API_URL` in `server.js` is `https://api.levelup99x.999online.ai` (production); override
with `LEVELUP_API_URL` env or the `apiUrl` argument to `levelup_save_config` for local dev
(`http://localhost:8787`). An `apiUrl` saved via `levelup_save_config` must be `https://` for any
host, or `http://` only for a loopback host (`localhost`, `127.0.0.1`, `[::1]`); changing an
existing `apiUrl` is called out in the tool result. `LEVELUP_CONFIG_PATH` overrides the config
file location (used by the self-check).

## Error mapping

| API response | Tool result |
| --- | --- |
| 401 | "token invalid or revoked — reconnect" (mint a new token in Settings, save via `levelup_save_config`) |
| 403 | "Jin is sleeping (Pro feature)" + upgrade pointer |
| 429 | rate-limited message surfacing `Retry-After` |
| network failure | readable message naming the API URL, never the token |
| other | status + the API's `{ error: { code, message } }` envelope |
