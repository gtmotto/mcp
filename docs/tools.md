# GTM Otto MCP: the 21 tools

Endpoint `https://agent.gtmotto.com/mcp` (Streamable HTTP). Server name `gtmotto`.
Read tools are safe to call at any time and most clients auto-approve them. Write
tools change something, spend something, or act from the user's LinkedIn account;
the ones marked **ask first** must only be called when the human asked for it in the
conversation.

Ids echo back from other tools' output: a `playId` comes from `gtm_list_plays`, an
`approvalId` from `gtm_list_approvals`. A wrong or foreign id reads as "not found".

Lanes are addressed by `(playId, key)`. `key` is one of `people_search`,
`competitor_posts` (Post discovery), `job_offers`, `account_list`. A fifth key,
`post_engagers`, is retired: it still runs on rows that exist, but put post URLs in
`competitor_posts.postUrls` on any new play.

## Read tools

### `gtm_whoami`

Who this session belongs to and what the workspace may do: org name, plan,
subscription status, product and play counts, the lead meter (leads added this
month, cap, remaining), and `next`: the step to follow (`sign_up`, `choose_plan`,
`create_play`, `connect_linkedin`, `preview`, `activate`, `approve_messages`,
`nothing_due`) with a `url` when the step needs a browser. Start here.

Arguments: none.

### `gtm_list_plays`

Every play with enough context to route on: product, active or paused,
`seatConnected` (false means the play cannot act whatever its status says), and the
spread of its leads across pipeline stages. Also carries `next`.

Arguments: none.

### `gtm_show_play`

One play in full: goal and voice, the account's daily cap and working hours (shared
by every play on the seat), the ICP facets, the five lanes with config and last run,
the autonomy ladder (which rungs run alone, which wait for a human), the LinkedIn
seat, today's used/cap for the seat and this play's share. Also carries `next`.

Arguments: `playId` (string).

### `gtm_play_stats`

The Stats screen: leads sourced, warmed, invited, new connections, conversations
started, over four windows (all time, 30 days, 7 days, 24 hours), each windowed
count paired with the window before as a percentage delta. "Conversations started"
counts real replies, never the openers GTM Otto sent. During the first two days after
activation it says "warming up, first invites expected on <date>" instead of zeros.

Arguments: `playId` (string).

### `gtm_linkedin_status`

The play's seat and today's headroom against LinkedIn's limits. States: `no_seat`,
`seat_broken` (disconnected or deleted; the queue is parked until a human
reconnects), `pending`, `classic`, `sales_navigator` (the search backend the ICP
compiles to).

Arguments: `playId` (string).

### `gtm_sources_reference`

The reference for the sourcing lanes: every lane, every config field with meaning,
default and cap, the fit, intent and post gates, what makes a lane runnable, the
5-minute tick and the 24-hour cadence, what a preview costs versus a run, and what
the surface will not do. No backend call. Read it before the first
`gtm_configure_source`, `gtm_set_source` or `gtm_run_source`.

Arguments: none.

### `gtm_show_sources`

All five lanes of a play, configured or not. Per lane: `configured`, `on`, parsed
`config` with defaults filled, the cockpit `summary` (null means needs setup),
`blocked` (`needs_config`, `icp_empty` or null), `lastRunAt`, `nextRunAt`, the last
run's funnel, `addedToday`, `failures`, and the lane's memory (`accounts[]` with
`found: true|false` per typed company on `account_list`, `harvestedPosts` on the post
lanes, cached `searches` on Post discovery). Play level: `state` (`no_sources`,
`paused`, `draft`, `no_seat`, `seat_broken`, `icp_empty`, `needs_config`,
`lead_quota`, `never_ran`, `no_results`, `provider_error`, `added_nothing`, `ok`),
provider, seat, ICP facet count, remaining lead quota.

Arguments: `playId` (string).

### `gtm_source_runs`

The run history of one lane, newest first: scanned, dropped by fit, dropped by
intent, added, duplicates, capped, the ICP facets LinkedIn could not filter on, and
the provider's status and message when it refused. Tells "the query returns nobody"
from "everyone was already a lead" from "LinkedIn is rate-limiting the seat".

Arguments: `playId` (string), `key` (lane key), `limit` (number, 1 to 50, default 10).

### `gtm_list_approvals`

The drafted messages waiting for a human: for each, the `approvalId`, the play, the
lead (name, title, company), the rung (`first_dm`, `follow_up`, `reply`, `comment`),
the drafted text and when it was drafted. Empty when nothing is waiting.

Arguments: `playId` (string, optional; omit for the whole workspace).

## Write tools

### `gtm_estimate_icp`

How big an audience a candidate ICP would reach, as "about N companies match",
before anything is created. Writes nothing: no play, no ICP row, no cache. Use it to
iterate: propose facets, see 40, widen the locations, see 900, then create the play.
Returns null when no data provider is configured, which is not an error. Works
without signing in.

Arguments: `industries` (string[]), `locations` (string[]), `sizes` (string[],
headcount bands such as `"11-50"`). All optional.

### `gtm_create_play`

Create a fully configured play in one call: the product (an existing `productId`, or
a `websiteUrl` that seeds one and starts its research), the play, the ICP facets, the
lanes to configure, the autonomy ladder and the caps. Draft by default.

Arguments: `name` (string, required); `productId` (string) or `websiteUrl` (string)
plus optional `productName`; `goal` (string); `voice` (string); `icp` (object with
`titles`, `seniority`, `languages`, `industries`, `sizes`, `locations`, `exclude`,
each string[]); `sources` (array of `{ key, on?, config? }`); `autonomy` (array of
`{ key, auto }` with keys `visit`, `like`, `comment`, `invite`, `first_dm`,
`follow_up`, `reply`; omitted rungs wait for a human); `dailyCap` (number, account
level, defaults to the seat's full allowance of 145); `workingHours`
(`{ start: "09:00", end: "18:00", tz: "Europe/Paris" }`, account level); `activate`
(boolean, default false; only starts the play if a live seat is already connected).
**Ask first** before passing `activate: true`.

### `gtm_update_icp`

Change the ICP a play runs: the query behind `people_search` and the fit gate on
every other lane. Plain words; GTM Otto normalises odd labels itself. Only the facets
you name change. The first write on a play forks its own ICP row, so no other play's
targeting is rewritten. Filling an empty ICP unblocks lanes parked on `icp_empty`.

Arguments: `playId` (string) plus any of `titles`, `seniority`, `languages`,
`industries`, `sizes`, `locations`, `exclude` (string[]).

### `gtm_update_play`

Change a play's own knobs and the account's pacing. `paused: true` is the stop
button; `paused: false` resumes. `status: "active"` launches a draft;
`status: "draft"` takes it back off the road. No live seat means neither launch nor
resume takes effect; the play stays a draft and the note says so. Does not touch the
autonomy ladder. **Ask first** before launching, resuming or raising the cap.

Arguments: `playId` (string); optional `name`, `goal`, `voice` (string); `dailyCap`
(positive number, account level); `workingHours` (`{ start, end, tz }`, account
level); `paused` (boolean); `status` (`"active"` or `"draft"`).

### `gtm_configure_source`

Write a lane's settings: upload an account list, set the Post discovery intent and
anchors, change the job keywords, move a gate. Merge semantics: only the fields you
send change. Lists (`companies`, `postUrls`, `competitors`) replace by default;
`listMode: "append"` adds to what is there, deduped case-insensitively. Caps: 100
companies, 25 posts, 25 anchors; what did not fit comes back in `truncated`. Unknown
fields are refused. Editing makes the lane due again at once, but nothing runs inside
the call.

Arguments: `playId` (string), `key` (lane key), `config` (object), `listMode`
(`"replace"` or `"append"`, default replace), `on` (boolean, optional; also flips the
switch).

`config` fields by lane:

- `people_search`: `keywords` (string), `fitGate`.
- `competitor_posts`: `intent` (string, 1000 chars max, any language), `competitors`
  (string[], anchors), `postUrls` (string[]), `minReactions` (number or null),
  `postGate` (0 to 100 or null), `commenters` (boolean, default true), `reactors`
  (boolean, default true), `intentGate` (0 to 100 or null), `fitGate`.
- `job_offers`: `keywords` (string, the role being hired), `postedWithinDays` (number
  or null), `fitGate`.
- `account_list`: `companies` (string[], names, websites or LinkedIn page URLs, max
  100), `fitGate`.
- `fitGate` on every lane: `"strict"`, `"standard"` (default), `"loose"`, `"off"`.

### `gtm_set_source`

Switch a lane on or off. Switching on a lane with nothing to run on (no companies, no
intent, anchors or posts, no role) is refused; configure it first. An empty ICP is
not a refusal: `people_search` may go on ahead of the ICP and reads `blocked:
icp_empty` until `gtm_update_icp` fills it. Switching off never fails. Nothing runs
inside this call.

Arguments: `playId` (string), `key` (lane key), `on` (boolean).

### `gtm_preview_source`

Dry-run one lane: "about N people match" and a 10-person sample, without inserting a
lead, spending the day's budget or moving the run clock. Same query, gates and
scorers as a real run, on up to 50 people. `size.precision` is `exact`, `atLeast` or
`unavailable` (`skipped` says why). Reports `unmapped` facets, `unresolved` values,
which sampled people are `alreadyALead`, and on `account_list` what each company
resolved to. Costs provider calls and, when a gate is on, model tokens. Refused on a
paused play.

Arguments: `playId` (string), `key` (lane key).

### `gtm_run_source`

Run one lane now, through the same code as the tick, with force: an off lane still
runs, a draft play still runs (that is how you try a lane before launch), a paused
play does not. Real: provider calls are made, leads are inserted, the daily budget
and monthly quota are spent. Returns the run's funnel and `skipped` plus `reason`
when it could not run. **Ask first.**

Arguments: `playId` (string), `key` (lane key).

### `gtm_forget_harvested_posts`

The tuning escape hatch for the post lanes. A post lane remembers every post it
already paid to harvest, so changing a gate and running again answers "no new posts".
This clears that memory so the next run judges today's posts against the new
settings. Every forgotten post is a fresh, paid engager fetch. Only
`competitor_posts` and `post_engagers` have this memory. **Ask first.**

Arguments: `playId` (string), `key` (lane key).

### `gtm_connect_linkedin`

Start (or restart, when the seat is `seat_broken`) the LinkedIn connection for a
play. Returns a hosted browser link; the human completes it in the browser, then the
agent polls `gtm_linkedin_status` until the state is `classic` or `sales_navigator`.
The only cockpit step the MCP needed and now has.

Arguments: `playId` (string).

### `gtm_approve`

Approve one drafted message by its approval id. GTM Otto sends it from the human's
account within working hours. The client asks the human before each call; show the
full text first. **Ask first, every time.**

Arguments: `approvalId` (string).

### `gtm_reject`

Reject one drafted message by its approval id. Nothing is sent; the lead stays in the
pipeline. The client asks the human before each call. **Ask first, every time.**

Arguments: `approvalId` (string).
