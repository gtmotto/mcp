# GTM Otto MCP

**Give Claude a LinkedIn seat that works while you sleep.**

GTM Otto is self-driving LinkedIn prospecting. From Claude, Cursor or ChatGPT you
describe who you want to meet; GTM Otto finds them daily, warms them up from your
own LinkedIn account (visit, like, comment), invites them, and opens the
conversation. Nothing is said without you.

This repository is the public home of the GTM Otto MCP server: the registry
manifest (`server.json`), the Claude Code plugin with eight skills, the tool
reference and the onboarding guide. The server itself runs at
`https://agent.gtmotto.com/mcp` (Streamable HTTP).

- Site: [gtmotto.com](https://gtmotto.com)
- Docs: [gtmotto.com/docs/mcp](https://gtmotto.com/docs/mcp)
- Connect page: [app.gtmotto.com/connect](https://app.gtmotto.com/connect)

**Release status (2026-09-22):** listed on the official MCP Registry as
[`com.gtmotto/linkedin`](https://registry.modelcontextprotocol.io/v0.1/servers/com.gtmotto%2Flinkedin/versions/1.2.1)
(manifest 1.2.1). `https://agent.gtmotto.com/mcp` answers `initialize` with no
credentials; OAuth is served at `/.well-known/oauth-protected-resource`
(authorization server `clerk.gtmotto.com`); 21 tools are deployed, including
`gtm_connect_linkedin` and the three approval tools.

## What it is, in one paragraph

Every other LinkedIn MCP is a remote control for a tool you still have to operate:
lists to upload, sequences to build, seats to rotate. GTM Otto is the only one where
the outreach keeps running after the chat ends. A **play** is an ICP plus up to five
**sourcing lanes** plus an autonomy level. The server exposes 21 tools and,
deliberately, no send verb: the agent can set up, size, preview, launch, pause and
review a play, and it can approve or reject one drafted message at a time, but it
cannot write to a stranger on your behalf.

## Install

The endpoint is the same everywhere: `https://agent.gtmotto.com/mcp`.

**Authentication.** Add the server with no credentials. The first time the agent
calls a protected tool, the client shows a **Connect** card; sign in (or sign up) in
the popup and the call is retried. Scripts and clients that cannot do OAuth can send
an API key instead: `Authorization: Bearer gtm_sk_...`, minted at
[app.gtmotto.com/connect](https://app.gtmotto.com/connect).

### Claude (web and desktop)

Settings, then Connectors, then **Add custom connector**. Name it `GTM Otto`, paste
`https://agent.gtmotto.com/mcp`, save. Start a chat and say what you want; the
Connect card appears on the first protected call.

### Claude Code

```bash
claude mcp add --transport http gtmotto https://agent.gtmotto.com/mcp
```

Then `/mcp` inside a session to authenticate in the browser. To use an API key
instead:

```bash
claude mcp add --transport http gtmotto https://agent.gtmotto.com/mcp \
  --header "Authorization: Bearer gtm_sk_..."
```

### Cursor

[![Install MCP Server](https://cursor.com/deeplink/mcp-install-dark.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=gtmotto&config=eyJ1cmwiOiJodHRwczovL2FnZW50Lmd0bW90dG8uY29tL21jcCJ9)
[![Install MCP Server](https://cursor.com/deeplink/mcp-install-light.svg)](cursor://anysphere.cursor-deeplink/mcp/install?name=gtmotto&config=eyJ1cmwiOiJodHRwczovL2FnZW50Lmd0bW90dG8uY29tL21jcCJ9)

The deeplink is
`cursor://anysphere.cursor-deeplink/mcp/install?name=gtmotto&config=eyJ1cmwiOiJodHRwczovL2FnZW50Lmd0bW90dG8uY29tL21jcCJ9`,
where the `config` value is the base64 of `{"url":"https://agent.gtmotto.com/mcp"}`.
By hand, add this to `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "gtmotto": { "url": "https://agent.gtmotto.com/mcp" }
  }
}
```

### ChatGPT

Settings, then Connectors, then **Create** (developer mode must be on). Name
`GTM Otto`, URL `https://agent.gtmotto.com/mcp`, authentication **OAuth**. Enable the
connector in a chat from the tools menu.

### Any other client

Any MCP client that speaks Streamable HTTP works: point it at
`https://agent.gtmotto.com/mcp`, with OAuth or the `Authorization` header above.
n8n, Windsurf, VS Code and the reference `mcp-remote` shim all fit.

## Skills for Claude Code

The `gtm-otto` plugin connects the server and adds eight skills that turn a sentence
into a running play:

```
/plugin marketplace add gtmotto/mcp
/plugin install gtm-otto@gtmotto
```

| Skill | What it does |
|---|---|
| `launch-a-play` | Estimate the ICP, create a draft play, connect LinkedIn, preview each lane, confirm, activate, explain the next 48 hours. |
| `icp-to-play` | A sentence, a URL or a persona into the seven ICP facets, a company count, and a `people_search` lane on a new draft or an existing play. |
| `post-discovery` | Turn "people complaining about X on LinkedIn" into a Post discovery lane, with a worked French example. |
| `hiring-signals` | "Companies hiring for X" into a Job offers lane that reaches the buyer inside each hiring company, not the recruiter. |
| `weekly-play-review` | Stats, run history and the approval queue folded into a one-page review with concrete changes. |
| `instantly-to-linkedin` | With the Instantly MCP also connected: companies that opened or replied to a campaign become an account list on LinkedIn. |
| `heyreach-migration` | From an exported HeyReach lead list: companies to an account list, facets to the ICP, and an honest note on what does not carry over. |
| `linkedin-safety-check` | Seat state, daily cap and working hours turned into a plain-English health report. |

Each skill names the exact tools and arguments it calls, what to show you, and where
it stops to ask.

## Example prompts

- "Find me CFOs at Swiss fintechs and set up a play, but do not launch it yet."
- "People on LinkedIn complaining about paying 100 euros a month for Sales Navigator. Build a Post discovery lane on that and show me a sample."
- "Take every company that replied to my Instantly campaign 'Q3 agencies' and add them to the LinkedIn play."
- "Review my play 'Fintech CFOs' for the week and tell me what to change."
- "Is my LinkedIn account safe at the current daily cap?"

## Tools

Full reference with arguments: [docs/tools.md](docs/tools.md).

| Tool | Scope | One line |
|---|---|---|
| `gtm_whoami` | read | Org, plan, lead meter, and the `next` step to follow. Start here. |
| `gtm_list_plays` | read | Every play with product, status, seat and pipeline spread. |
| `gtm_show_play` | read | One play in full: ICP, lanes, autonomy ladder, cap, working hours, seat. |
| `gtm_play_stats` | read | The pipeline over four windows, each with its delta versus the window before. |
| `gtm_linkedin_status` | read | The play's seat state and today's headroom against LinkedIn's limits. |
| `gtm_sources_reference` | read | The reference for every sourcing lane. Read before configuring one. |
| `gtm_show_sources` | read | All five lanes of a play: config, switch, blockers, last run, memory. |
| `gtm_source_runs` | read | Run history of one lane, newest first. |
| `gtm_list_approvals` | read | Drafted messages waiting for a human, with their approval ids. |
| `gtm_estimate_icp` | write | "About N companies match" for a candidate ICP. Writes nothing. |
| `gtm_create_play` | write | Create a fully configured play. Draft by default; `activate` flag. |
| `gtm_update_icp` | write | Change a play's seven ICP facets. Only the ones you name. |
| `gtm_update_play` | write | Name, goal, voice, `paused`, `status` active or draft, `dailyCap`, `workingHours`. |
| `gtm_configure_source` | write | Write a lane's settings (merge). Can switch it on in the same call. |
| `gtm_set_source` | write | Switch a lane on or off. Refused when there is nothing to run on. |
| `gtm_preview_source` | write | Dry-run one lane: size and a 10-person sample. No side effects. |
| `gtm_run_source` | write | Run one lane now. Real leads, real budget. Only when asked. |
| `gtm_forget_harvested_posts` | write | Clear a post lane's memory so a settings change can be re-tested. Costs. |
| `gtm_connect_linkedin` | write | Returns a browser link to connect the LinkedIn seat; then poll `gtm_linkedin_status`. |
| `gtm_approve` | write | Approve one drafted message by approval id. The client asks you first. |
| `gtm_reject` | write | Reject one drafted message by approval id. The client asks you first. |

## What it deliberately does not do

- **No send verb.** No invite, DM, comment or like tool. The agent configures the
  play; GTM Otto runs it from your account on its own cadence; anything that speaks
  waits for your approval, one message at a time, in the chat or in the cockpit.
- **Nothing runs inside a write.** Configuring a lane is not switching it on, and
  switching it on does not run it. A lane runs once per 24 hours on the 5-minute
  tick, when the play is active and not paused. `gtm_preview_source` is free of side
  effects; `gtm_run_source` spends real budget and is only called when you asked.
- **No launching without you.** The agent never activates a play, resumes a paused
  one, or runs a lane unless you asked for it in the conversation. Activation acts
  from your own LinkedIn account.
- **No lead browsing, no autonomy changes after creation, no delete.**
- **No list of people.** The server takes companies (`account_list`, at most 100)
  and post URLs, never a CSV of individual LinkedIn profiles.
- **Warm-up is real.** The first invites land about two days after activation.
  The silent days are the product working, not a bug.

## Repository layout

```
server.json                          registry manifest (com.gtmotto/linkedin)
.claude-plugin/marketplace.json      Claude Code marketplace "gtmotto"
plugins/gtm-otto/                    the plugin: MCP server + eight skills
  .claude-plugin/plugin.json
  skills/<name>/SKILL.md
docs/tools.md                        the 21 tools with arguments
docs/onboarding.md                   the ideal first conversation
CHANGELOG.md
```

## Links

- Setup guides per client: [gtmotto.com/docs/mcp](https://gtmotto.com/docs/mcp)
- Official MCP Registry entry: `com.gtmotto/linkedin`
- Agent Skills format: [agentskills.io/specification](https://agentskills.io/specification)
- Claude Code plugins: [code.claude.com/docs/en/plugins](https://code.claude.com/docs/en/plugins)

## The free skills

The playbooks that need nothing but LinkedIn (find buyers in comments, ICP to a
Sales Navigator search, hiring-signal prospecting by hand, LinkedIn limits, the
comment and invite writer) live in [gtmotto/gtm-skills](https://github.com/gtmotto/gtm-skills).
That repository registers no server and needs no account; each of its skills ends
with a pointer to the skill here that runs the same recipe daily.

## Contributing

This repository is **mirrored out of the GTM Otto monorepo**. Issues and pull
requests are welcome here and are read; fixes are applied upstream and land in the
next mirror, so a PR may be closed with a "merged upstream" note rather than merged
in place. Keep prose free of em dashes, and never describe the product as an
automated sales rep of any kind: it is self-driving LinkedIn prospecting.

## License

MIT. See [LICENSE](LICENSE).
