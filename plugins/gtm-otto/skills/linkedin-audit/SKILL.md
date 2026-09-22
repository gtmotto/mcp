---
name: linkedin-audit
description: Score any LinkedIn profile on two axes, how active it is and how weird it reads, and place it in one of four quadrants (invisible, occasional bot, operator, spray). Needs no account and no GTM Otto connection; the agent reads the public profile with whatever tools it has (a LinkedIn data MCP, a browser, web fetch and search) and scores it with the rules in this skill. If the human's GTM Otto seat is connected, adds the signals a stranger cannot see (pending invites, acceptance rate, template messages). Use when the user asks how good they are at LinkedIn, whether they look like a bot, wants to audit a profile, a competitor or a colleague, asks for a LinkedIn score, or says their outreach is not working.
license: MIT
compatibility: Works standalone in any agent that can read public web pages or has a LinkedIn data tool. The full audit is optional and needs the GTM Otto MCP server (https://agent.gtmotto.com/mcp) with a LinkedIn seat connected.
metadata:
  author: gtmotto
  version: "1.0"
---

# LinkedIn audit

Two questions about one account, answered with numbers:

- **X, activity.** How much of LinkedIn does this account actually use? Posting,
  commenting, reacting, inviting, messaging, following up. 0 to 100.
- **Y, weirdness.** How much of that activity reads as automated? Generic
  comments, bursts, off-hours, template messages, blank invites at scale,
  hundreds of pending invites. 0 to 100.

Together they place the account in one of four quadrants:

| | Not weird (Y < 40) | Weird (Y ≥ 40) |
|---|---|---|
| **Not active** (X < 50) | **A. Invisible.** Safe, and wasted. Most accounts. | **B. Occasional bot.** Rarely shows up, and when it does it reads automated. Worst risk for the least result. |
| **Active** (X ≥ 50) | **C. Operator.** Every day, sounds like a person. The target. | **D. Spray.** Loud, and one warning away from a restriction. |

The job of this skill is to say which quadrant, why, and what moves the account
to C. The agent does the reading and the scoring itself; nothing here requires
GTM Otto.

## Two tiers, one report

| | Public audit | Full audit (optional) |
|---|---|---|
| Needs | a profile URL, and any way to read public LinkedIn pages | the human signed in to GTM Otto with a seat connected |
| Data | posts, comments left on others' posts, reactions, profile fields, timestamps | everything public, plus invites sent, pending invites, acceptance rate, messages sent, reply rate, template similarity, hours, network |
| Cannot see | anything LinkedIn restricts accounts for | what other tools or the human did by hand outside the seat |

The public audit runs on anyone: the human's own profile, a competitor, a
colleague, a prospect. The full audit only ever runs on the human's own account.

## Stop rules

- Read-only. This skill writes nothing on LinkedIn, sends nothing, and changes
  no play. If GTM Otto is connected, the only actions it may propose are
  `gtm_update_play` with a lower `dailyCap` or tighter `workingHours`, and
  `gtm_create_play` as a draft (never `activate: true`), each after a yes.
  Withdrawing pending invites is proposed as a by-hand step.
- Audit behaviour, never the person. "12 of the last 30 comments do not
  reference the post", not "this person is a spammer". No adjectives about the
  human behind a public profile.
- Never invent a number. A signal the sources did not return is reported as
  "not visible" in its row, and the axis is scored out of the points that were
  visible. Say which source produced each number if the human might act on it.
- Never ask the human to paste their messages, comments or exports. If nothing
  can read the profile, say so and stop (see step 2).
- Reading a profile with a logged-in browser counts as the human's own LinkedIn
  activity. Do it at a human pace: a handful of page loads, no scrolling loops.

## Steps

1. **Get the URL.** Accept `linkedin.com/in/<slug>` only. A company page, a
   post URL or a Sales Navigator URL is not a profile; say so and ask again. If
   the human gives a name instead, search the web for the profile, show the
   match, and confirm before reading anything.

2. **Read the public profile with what you have**, in this order, stopping at
   the first that works. Aim for the last 30 posts, the last 50 comments the
   account left on other people's posts (with the parent post's text), and
   reactions if the source exposes them, all with timestamps.

   | Source | How |
   |---|---|
   | A LinkedIn data tool in the client (Deepline `harvestapi_get_profile`, `harvestapi_get_profile_posts`, `harvestapi_get_profile_comments`; Apify LinkedIn actors; Linked API; any "profile posts" and "profile comments" tool) | Call profile, posts (last 3 months), comments (last month, then a second page if the first is full). Comments usually carry the parent post's excerpt. |
   | A browser tool, logged in to LinkedIn | Open `linkedin.com/in/<slug>/recent-activity/all/`, then `/recent-activity/comments/`. Read what is on screen, load at most two more screens each. |
   | Web fetch and search only | Fetch the profile URL; LinkedIn often returns a login wall. Then search `site:linkedin.com/posts "<name>"` and `site:linkedin.com "<name>" commented`, and fetch the post pages that come back; each one gives a post or a comment with a date. This is thin; say so in the report. |
   | Nothing above works | Say the profile could not be read from here, name what would (a LinkedIn data tool, a logged-in browser, or the full audit through GTM Otto), and stop. No score from zero data. |

   Normalise whatever came back into: a list of posts (text, date), a list of
   comments (text, date, parent post text and date), a reaction count if any,
   and the profile fields (banner present, headline, about text, location).

3. **Score X and Y with the tables below.** Do the arithmetic in the open so
   the human can check it: every signal line in the report shows its raw number.
   Judge "generic comment" by reading each comment against its parent post; a
   comment is generic when nothing in it could only have been written about
   that post.

4. **Full audit, only if available.** If the GTM Otto MCP is connected, call
   `gtm_whoami`; if signed in and `gtm_linkedin_status` shows `classic` or
   `sales_navigator` for a play, call `gtm_audit_seat` and merge its signals
   into the same tables (they carry the extra rows: invites, pending, acceptance,
   messages, templates, ceilings). If GTM Otto is not connected, skip this step
   silently; do not tell the human to connect anything before the report.

5. **Name the quadrant and the move.**
   - **A, invisible**: nothing to fix, everything to start. The move is activity
     without weirdness: a daily routine of visits, reactions, one or two specific
     comments, invites with a reason. Show what a not-weird week looks like in
     numbers.
   - **B, occasional bot**: fix weirdness before adding volume. Name the exact
     habits (the burst days, the comment template, the blank invites), then the
     routine.
   - **C, operator**: say so, name the one signal closest to a threshold, and
     offer nothing unless asked.
   - **D, spray**: risk first. Lower the volume, narrow the hours, withdraw stale
     pending invites, stop the template. Only then talk about results.
   Scores within 5 points of a threshold are "on the line": say so instead of
   pretending the quadrant is settled.

6. **Write the report.** Template below. Then, if there is an action, the first
   one with its exact step, then wait.

## How the scores are built

**X, activity** (0 to 100). Each signal is scaled against a full month for one
account at safe limits (the `linkedin-limits-and-safety` ceilings: 20 invites,
40 messages, 40 visits, 30 reactions, 15 comments a day, weekdays), so 100 means
"uses the whole allowance, every working day". Points for a signal = weight ×
min(1, observed / target).

| Signal | Target for full points | Public | Full | Weight |
|---|---|---|---|---|
| Posts in the last 30 days | 12 | yes | yes | 15 |
| Comments left on others' posts, last 30 days | 60 | yes | yes | 20 |
| Reactions given, last 30 days | 200 | yes | yes | 10 |
| Profile complete: banner present, headline says what you do for whom (not just a title), about section that is not a CV | 3 of 3 | yes | yes | 10 |
| Invites sent, last 30 days | 200 | | yes | 15 |
| First messages sent, last 30 days | 100 | | yes | 15 |
| Follow-up discipline: share of conversations with a second message from this account after silence | 80% | | yes | 10 |
| Network used: share of first-degree connections messaged at least once | 30% | | yes | 5 |

Public audits are scored out of the points visible (55 when reactions are
visible, 45 when not) and rescaled to 100; the report says which.

**Y, weirdness** (0 to 100). Each signal is a share or a count, capped at its
weight. Points = weight × min(1, observed / "full weird").

| Signal | Full weird at | Public | Full | Weight |
|---|---|---|---|---|
| Generic comments: share of comments that reference nothing in the parent post ("great insight", "love this", emoji-led, a restated headline) | 60% | yes | yes | 20 |
| Bursts: days with 5+ comments or reactions inside 30 minutes, last 30 days | 4 days | yes | yes | 10 |
| Off-hours: share of activity outside 7:00 to 22:00 in the profile's location | 40% | yes | yes | 10 |
| Stale-post comments: comments on posts older than 30 days at the time of commenting | 30% | yes | yes | 5 |
| Template posts: share of posts with near-identical structure to another post | 50% | yes | yes | 5 |
| Pending invites older than 3 weeks, not withdrawn | 300 | | yes | 15 |
| Acceptance rate on the last 100 invites (points rise as the rate falls below 30%) | 10% | | yes | 10 |
| Blank invites at scale: 50+ invites in 30 days with no note | yes/no | | yes | 5 |
| Template messages: share of first messages in a near-duplicate cluster (same text, names swapped) | 60% | | yes | 15 |
| Above the ceilings: days over 20 invites or 40 messages, last 30 days | 8 days | | yes | 5 |

Public audits score Y out of the 50 visible points and rescale to 100.
Thresholds: active at X ≥ 50, weird at Y ≥ 40, both deliberately blunt.

## The report

```
LinkedIn audit, <name or URL>, <date>            <public | full>, read via <source>

Activity   X = <n>/100   <not active | active | on the line>
Weirdness  Y = <n>/100   <not weird | weird | on the line>
Quadrant   <A invisible | B occasional bot | C operator | D spray>

Behind X
  <signal>: <raw number> → <points>/<weight>, <one line>
  <signal>: <raw number> → <points>/<weight>, <one line>
  <signal>: <raw number> → <points>/<weight>, <one line>
Behind Y
  <signal>: <raw number> → <points>/<weight>, <one line>
  <signal>: <raw number> → <points>/<weight>, <one line>
  <signal>: <raw number> → <points>/<weight>, <one line>
Not visible   <public audits: pending invites, acceptance rate, messages, templates>

The move to C
  1. <change>: <by hand, or the exact step>
  2. ...
A not-weird week on this account
  <n> visits, <n> reactions, <n> specific comments, <n> invites with a reason, <n> follow-ups, weekdays, <hours> <tz>
```

Under 30 lines. Numbers on every line that has one. No adjectives about the person.

## What to show the human

- The report, then the first proposed action, then wait.
- Public audit on their own profile: end with one line naming what a stranger
  cannot see ("pending invites and message templates are the two signals
  LinkedIn restricts for; they are only visible from inside the account") and,
  only if the GTM Otto MCP is present in the client, the `gtm_connect_linkedin`
  step. Once. If it is not present, do not mention GTM Otto at all.
- Public audit on someone else: the report only.
- If they ask for the score alone: the three header lines, nothing else.

## What this skill does not know

The public audit sees what LinkedIn shows a stranger, through whatever source
read it: no messages, no invites, no acceptance rate, and only the comments and
posts LinkedIn kept public. Search-only reads are the thinnest and can miss most
comments; the report must say so. The full audit sees what went through the GTM
Otto seat and what LinkedIn exposes for the account, so activity from other
tools or done by hand shows up in public signals but not in the invite and
message rows, which are therefore a floor.
