---
name: linkedin-safety-check
description: Read a GTM Otto play's LinkedIn seat state, today's headroom, the account's daily cap and working hours, and turn them into a plain-English health report with what to lower. Use when the user asks whether their LinkedIn account is safe, worries about limits, restrictions or bans, asks what the daily cap should be, or after a seat shows seat_broken.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# LinkedIn safety check

Everything GTM Otto does happens from the human's own LinkedIn account, so the
account's health is the whole business. This skill reads, judges, and proposes
lowering. It never raises anything on its own.

## Stop rules

- Reads only until the human approves a change. The only writes this skill may
  propose are `gtm_update_play` with a lower `dailyCap`, tighter `workingHours`, or
  `paused: true`, and `gtm_connect_linkedin` for a broken seat.
- Never raise `dailyCap` or widen `workingHours` unless the human asked in this
  conversation, and even then state the risk first.
- Never resume a paused play as part of a safety check.

## Steps

1. **Pick the play.** `gtm_list_plays`. The cap and hours are account-level, shared
   by every play on the same seat, so list every play on that seat and sum their
   activity.
2. **Seat state.** `gtm_linkedin_status` with `playId`. Read the state and today's
   headroom.
   - `no_seat`: nothing can act. Offer `gtm_connect_linkedin`.
   - `seat_broken`: the account was disconnected or deleted; the queue is parked.
     Ask whether LinkedIn showed a warning or a verification prompt before the
     disconnect. Offer `gtm_connect_linkedin` to reconnect, and propose a lower cap
     for the first week back.
   - `pending`: connection in progress; wait.
   - `classic` or `sales_navigator`: proceed.
3. **Pacing.** `gtm_show_play` with `playId`. Read `dailyCap` (default 145, the
   seat's full allowance), `workingHours` with its `tz`, today's used versus cap, and
   this play's share. Read the autonomy ladder: which rungs run alone.
4. **Recent load.** `gtm_play_stats` for the 7-day and 24-hour windows: invited and
   warmed counts. `gtm_show_sources` for `addedToday` per lane and any `failures`
   with a provider back-off, which is often LinkedIn pushing back.
5. **Judge.** Apply these rules, in this order:
   - Account under 3 months old, or under 500 connections: recommend a cap of 40 to
     60 and hours of 9 to 18 local.
   - Seat reconnected within the last 7 days: cap 60 for the week, whatever it was.
   - Invites above 100 per week on a classic account, or above 200 on Sales
     Navigator: recommend lowering the cap so invites stay under those lines.
   - Working hours covering weekends or more than 10 hours a day: recommend 9 to 18,
     Monday to Friday, in the account owner's time zone.
   - Several plays on one seat summing near the cap every day: say the cap is shared
     and propose a lower cap or pausing the weakest play (from its 7-day
     conversations).
   - Provider back-offs in two or more lanes: LinkedIn is rate-limiting the seat.
     Propose halving the cap for 3 days.
   - Nothing above triggered: say the account is within GTM Otto's limits, and name
     the one number that would change that first.
6. **Write the report.** See the template.
7. **Propose, then act on a yes.** For each recommendation, name the exact call:
   `gtm_update_play` with `playId` and `dailyCap: <n>`, or
   `workingHours: { start, end, tz }`, or `paused: true`. One at a time. Confirm the
   result. Remind that `dailyCap` and `workingHours` apply to every play on the seat.

## The report

```
LinkedIn safety, <account or play name>, <date>

Seat        <state>, <classic or sales_navigator>
Today       <used>/<cap> actions, hours <start>-<end> <tz>
This week   invited <n>, warmed <n>, connections <n>
Lanes       <n> on, <n> with provider back-off

Verdict     <one of: healthy / running hot / at risk / cannot act>
Why         <one or two lines from the rules above>

What to lower
  1. <change>, from <old> to <new>: <tool call>
  2. ...
What to leave
  <one line>
```

Under 25 lines. Plain words, no scores.

## What to show the human

- The report, then the first proposed change with its exact call, then wait.
- If the seat is broken: the reconnect link first, the report second.

## What this skill does not know

GTM Otto sees its own actions, not what the human does by hand on LinkedIn or
through other tools on the same account. If they run anything else on this seat,
say that the numbers here are a floor, and recommend a lower cap than the rules
give.
