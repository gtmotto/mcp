# The first conversation

This is what a good first session with GTM Otto looks like, from an empty client to a
play running on your own LinkedIn. It takes one conversation and three short trips to
the browser, all launched from the chat. There is no key to mint, no settings page,
no paste.

The rule behind every step: the agent always has one tool that works, and every
answer tells it the next step. You never hit a dead end, and the agent never guesses.

## Before you start

Add `https://agent.gtmotto.com/mcp` to your client (see the README for Claude,
Claude Code, Cursor and ChatGPT). Connecting and listing tools needs no account.

## The conversation

**You:** "Find me CFOs at Swiss fintechs."

**The agent sizes it first.** It calls `gtm_estimate_icp` with
`industries: ["Fintech"]`, `locations: ["Switzerland"]` and answers with something
like "about 1,240 companies match". This works before you sign in, so the first thing
you see is a number, not a login form. If the number is tiny, the agent widens a
facet and asks again; if it is huge, it narrows.

**The agent creates a draft.** It calls `gtm_create_play` with a name, the ICP
(`titles: ["CFO", "Chief Financial Officer"]`, the industry, the location), the
`people_search` lane switched on, and no `activate` flag. Nothing is live yet.

**Trip 1: sign in.** That call is protected, so the client shows a **Connect** card
in the chat. You sign up or sign in in the popup. If the workspace has no plan yet,
you pick one (7-day trial, card at checkout). The call is retried with your new
session and the draft play comes back with its id.

**The agent asks for a LinkedIn seat.** It calls `gtm_connect_linkedin` with the
play id and shows you the link.

**Trip 2: connect LinkedIn.** You open the link, connect your own account in the
browser, come back and say "done". The agent polls `gtm_linkedin_status` until it
reads `classic` or `sales_navigator` and tells you which search backend your ICP
will compile to, and today's headroom.

**The agent shows you who it would find.** For each lane that is on it calls
`gtm_preview_source`: "about 380 people match" and a 10-person sample with name,
title and company. This is free of side effects. You look at the sample and say what
is off: "too many controllers, I want the CFO only", "add Liechtenstein". The agent
changes the ICP with `gtm_update_icp` or the lane with `gtm_configure_source` and
previews again. Two or three rounds is normal.

**The agent asks: launch?** It states what launching means: the daily cap (145
actions by default, shared by every play on this account), the working hours, which
rungs run on their own (visit, like, invite) and which wait for you (anything that
speaks), and that the play acts from your own account.

**You:** "Go."

**The agent launches.** `gtm_update_play` with `status: "active"`. Only now, and only
because you said so.

**The agent explains the next 48 hours.** From `gtm_show_play`: the first day is
sourcing and warm-up (profile visits and likes), the first invites land around day
two, and the first drafted messages will wait for your approval. It tells you where:
in the chat with `gtm_list_approvals`, or in the cockpit at app.gtmotto.com.

Total: three browser trips (account, LinkedIn, and later approvals), all opened from
the chat, and one word to launch.

## What happens next

- **Days 1 and 2.** `gtm_play_stats` says "warming up, first invites expected on
  <date>" instead of showing zeros. The silence is the product working.
- **Day 3 onward.** Invites go out within the cap and working hours. New connections
  produce drafted openers.
- **Trip 3: approvals.** "Show me what is waiting" runs `gtm_list_approvals`. The
  agent shows each draft in full and asks before every `gtm_approve` or
  `gtm_reject`. Nothing is said without you.
- **Weekly.** "Review the play" runs the `weekly-play-review` skill: stats, run
  history, the approval queue, and concrete changes.

## If something is missing

The server answers with instructions, never walls:

| Situation | What the agent sees | What happens |
|---|---|---|
| Not signed in | 401 with an OAuth challenge on the first protected call | Connect card in the chat |
| No plan yet | `next.step = "choose_plan"` with the checkout link | The agent gives you the link |
| No LinkedIn seat | `next.step = "connect_linkedin"` | The agent calls `gtm_connect_linkedin` |
| Seat disconnected | `gtm_linkedin_status` reads `seat_broken` | Same tool reconnects; the queue is parked until then |
| Lane cannot run | `blocked: needs_config` or `icp_empty` | The agent fills what is missing, then previews |
| Messages waiting | `next.step = "approve_messages"` with the count | The agent offers to walk through them |

## The three things the agent will never do on its own

1. Activate a play or resume a paused one.
2. Run a lane for real (`gtm_run_source`). Previews are free; runs spend budget.
3. Approve or reject a message without showing it to you and asking.

Each of these happens only when you asked for it in the conversation.
