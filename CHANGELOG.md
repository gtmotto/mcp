# Changelog

All notable changes to this repository are recorded here. The MCP server itself
is versioned in `server.json`; the Claude Code plugin in
`plugins/gtm-otto/.claude-plugin/plugin.json`.

## 1.2.1 (2026-09-22)

Manifest-only release. Published to the official MCP Registry on 2026-09-22.

- **Registry name is now `com.gtmotto/linkedin`** (was `com.gtmotto/gtmotto`).
  The registry's `search` parameter matches the server *name* string only — a
  word that appears in a description but not the name returns zero hits — so the
  previous name was reachable only by people who already knew the brand.
  `linkedin` is the category query, with 62 servers on it. The `com.gtmotto`
  namespace and the `GTM Otto` display title are unchanged.
- `description` rewritten within the 100-character schema cap to name the three
  daily actions and the approval gate, instead of listing client names that
  every server in the registry shares.
- `websiteUrl` carries `?utm_source=mcp-registry`.
- `_meta.io.modelcontextprotocol.registry/publisher-provided` sets the GitHub
  subregistry display name.
- Namespace proved by DNS: a `v=MCPv1; k=ed25519` TXT record on the apex of
  gtmotto.com. Publishing a new version re-authenticates with that key.

## Plugin 1.1.0 (2026-09-18)

Two skills moved in from `gtmotto/gtm-skills`, which is now the account-free
repository (its `.mcp.json` and its three MCP-only skills are gone; each of its
five playbooks points here for the daily lane).

- `hiring-signals`: "companies hiring for X" into a `job_offers` lane, ICP pointed
  at the buyer inside the hiring company, preview, tune, switch on.
- `icp-to-play`: seven facets from a sentence or URL, `gtm_estimate_icp`, then
  `gtm_update_icp` on an existing play or a new draft with `people_search` on.
- Plugin manifests say eight skills. The server is unchanged at 1.2.0.

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
