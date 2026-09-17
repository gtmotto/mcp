---
name: weekly-play-review
description: Produce a one-page weekly review of a GTM Otto play from its stats, its lanes' run history and the approval queue, ending in concrete changes the human can approve one by one. Use when the user asks how a play is doing, wants a weekly or monthly review, asks what to change, or says the play feels slow or noisy.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# Weekly play review

Replaces the campaign-dashboard habit. Three reads, one page, and a list of changes
the human says yes or no to. The review itself changes nothing.

## Stop rules

- Reads only until the human approves a change. Never call `gtm_update_play`,
  `gtm_update_icp`, `gtm_configure_source`, `gtm_set_source`,
  `gtm_forget_harvested_posts` or `gtm_run_source` before a yes on that specific
  change.
- Never resume a paused play or raise the daily cap on your own initiative.
- Never approve or reject a message without showing its full text and asking.

## Steps

1. **Pick the play.** `gtm_list_plays`. If several, ask; if the human said "all",
   run the review per play and rank them by conversations started in 7 days.
2. **Stats.** `gtm_play_stats` with `playId`. Take the 7-day window and its delta
   versus the week before for each stage: sourced, warmed, invited, new connections,
   conversations started. If it says "warming up", the play is under two days old:
   say so and stop after step 4; there is nothing to tune yet.
3. **Lanes.** `gtm_show_sources` with `playId`. For each lane that is on, call
   `gtm_source_runs` with `playId`, `key`, `limit: 7`. Sum scanned, dropped by fit,
   dropped by intent, added, duplicates, capped over the week. Note `failures`,
   `blocked`, the provider status of any refused run, and `unmapped` facets.
4. **Seat and pacing.** `gtm_show_play` for the daily cap, working hours, the seat
   state and today's used versus cap. `gtm_linkedin_status` if the seat is anything
   but `classic` or `sales_navigator`.
5. **Approvals.** `gtm_list_approvals` with `playId`. Count what is waiting and how
   old the oldest draft is.
6. **Diagnose.** Match the numbers to a cause:
   - Sourced low, scanned high, added low: the fit gate or intent gate is dropping
     most people. Loosen one gate, or fix an `unmapped` facet in the ICP.
   - Scanned low: the query is narrow or the provider refused (check status). Widen
     a facet, add anchors, or wait out a back-off.
   - Duplicates high: the lane has exhausted its query. Add a lane or change the
     intent.
   - Invited high, connections low: the audience is wrong for the account, or the
     invite note needs work (voice on the play).
   - Connections up, conversations flat: drafts are sitting in approvals. Walk
     through them.
   - `capped` every day: the cap is the bottleneck; only raise it if
     `linkedin-safety-check` says the account has headroom.
7. **Write the page.** See the template below.
8. **Propose changes, one at a time.** For each, name the tool and the exact
   arguments, and ask. On yes, make the call and confirm the result. On no, move on.
9. **Offer the approval walk.** If drafts are waiting, show each in full and ask
   before `gtm_approve` or `gtm_reject` with its `approvalId`.

## The page

```
Play: <name>   Week of <date>   Seat: <state>   Cap: <used>/<cap>, <hours> <tz>

Pipeline, last 7 days (vs. week before)
  sourced <n> (<delta>)  warmed <n> (<delta>)  invited <n> (<delta>)
  connections <n> (<delta>)  conversations <n> (<delta>)

Lanes
  people_search    scanned <n>  added <n>  dropped fit <n>  dup <n>  <note>
  competitor_posts scanned <n>  added <n>  dropped intent <n>  <note>
  ...

Waiting for you: <n> drafts, oldest <age>

What I would change
  1. <change>: <tool>(<arguments>). Why: <one line>.
  2. ...

What I would leave alone
  <one line>
```

Keep it under 40 lines. Percentages only where the base is above 20.

## What to show the human

- The page, then the first proposed change with its exact call, then wait.
- Never the raw JSON; the numbers, in a fixed order, so week-over-week reads at a
  glance.

## Edge cases

- `state: paused`: report it first. Ask whether they want it resumed; only then
  `gtm_update_play` with `paused: false`.
- `state: seat_broken`: nothing ran. Offer `gtm_connect_linkedin` and stop the
  review there.
- `state: lead_quota`: the month's quota is spent. Say when it resets (from
  `gtm_whoami`) and do not propose lane changes.
- A lane with `failures` and a provider back-off: propose nothing for that lane
  this week; the back-off clears on its own.
