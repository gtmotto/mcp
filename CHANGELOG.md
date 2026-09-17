# Changelog

All notable changes to this repository are recorded here. The MCP server itself
is versioned in `server.json`; the Claude Code plugin in
`plugins/gtm-otto/.claude-plugin/plugin.json`.

## 1.2.0 (2026-09-17)

First public release of `gtmotto/mcp`.

- `server.json` for the official MCP Registry: `com.gtmotto/gtmotto`, Streamable
  HTTP at `https://agent.gtmotto.com/mcp`, OAuth by default with an optional
  `Authorization: Bearer gtm_sk_...` header for scripts.
- Claude Code plugin marketplace (`/plugin marketplace add gtmotto/mcp`) with one
  plugin, `gtm-otto`, that connects the server and ships six skills:
  `launch-a-play`, `post-discovery`, `weekly-play-review`, `instantly-to-linkedin`,
  `heyreach-migration`, `linkedin-safety-check`.
- `docs/tools.md`: the 21 tools with scope, one-liner and arguments.
- `docs/onboarding.md`: the ideal first conversation, from an empty client to a
  live play on your own LinkedIn.
- README with per-client install (Claude web and desktop, Claude Code, Cursor
  deeplink, ChatGPT, any Streamable HTTP client).
