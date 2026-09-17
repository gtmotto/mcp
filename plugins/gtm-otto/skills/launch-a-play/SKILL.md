---
name: launch-a-play
description: Turn a one-sentence audience into a live GTM Otto play on LinkedIn, in one conversation. Estimate the ICP, create a draft play, connect the LinkedIn seat, preview each sourcing lane, confirm with the human, activate, explain the next 48 hours. Use when the user describes who they want to meet on LinkedIn, asks to start prospecting, set up a play, or launch outreach.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# Launch a play

A play is an ICP plus up to five sourcing lanes plus an autonomy ladder. This skill
takes the human from a sentence to an active play without a single guess. Every
GTM Otto answer carries `next`; follow it when in doubt.

## Stop rules

- Never pass `activate: true` to `gtm_create_play`, never call `gtm_update_play` with
  `status: "active"` or `paused: false`, and never call `gtm_run_source`, unless the
  human asked for exactly that in this conversation.
- Never call `gtm_configure_source` before reading `gtm_sources_reference` once in
  the session.
- Preview (`gtm_preview_source`) is free of side effects; a run is not. Use previews
  to tune.
- If a call returns `next.url`, give the human the link and wait. Do not retry the
  call blindly.

## Steps

1. **Orient.** Call `gtm_whoami`. Note the plan, the lead meter and `next.step`. If
   `next.step` is `sign_up` or `choose_plan`, continue anyway: the estimate works
   without an account and the Connect card appears on the first protected call.
2. **Size the audience.** Extract facets from the sentence and call
   `gtm_estimate_icp` with `industries`, `locations`, `sizes`. Show the human
   "about N companies match". Under 50: propose widening one facet. Over 5,000:
   propose narrowing. Iterate until the human is satisfied with the number.
3. **Create the draft.** Call `gtm_create_play` with:
   - `name`: short, the audience in three words.
   - `productId` from `gtm_whoami` if a product exists, else `websiteUrl` (ask for
     the company website if unknown).
   - `goal` and `voice` in the human's words.
   - `icp`: `titles`, `seniority`, `languages`, `industries`, `sizes`, `locations`,
     `exclude`, plain words, only the facets the human gave.
   - `sources`: `[{ key: "people_search", on: true }]` at minimum. Add
     `competitor_posts` with an `intent` if the human described a pain, `job_offers`
     with `keywords` if they named a role being hired, `account_list` with
     `companies` if they named companies.
   - No `activate`. Leave `autonomy` unset unless the human asked; omitted rungs wait
     for a human, which is the safe default.
   Show the returned `playId`.
4. **Connect LinkedIn.** Call `gtm_linkedin_status` with the `playId`. If `no_seat`
   or `seat_broken`, call `gtm_connect_linkedin` with the `playId`, show the link,
   and say "open this, connect your own account, then tell me done". When they say
   done, call `gtm_linkedin_status` again; repeat up to three times a minute apart.
   Report the state (`classic` or `sales_navigator`) and today's headroom.
5. **Read the lanes.** Call `gtm_sources_reference` once, then `gtm_show_sources`
   with the `playId`. Any lane with `blocked: needs_config` or `icp_empty` needs a
   fix before it can run: `gtm_update_icp` for the ICP, `gtm_configure_source` for
   the lane's fields.
6. **Preview each lane that is on.** For each, call `gtm_preview_source` with
   `playId` and `key`. Show: `size` with its precision, the 10-person sample as a
   table (name, title, company, `alreadyALead`), and the `unmapped` facets. Ask
   "does this look like who you want to meet?". Adjust with `gtm_update_icp` or
   `gtm_configure_source` and preview again. Two or three rounds is normal.
7. **Confirm.** Call `gtm_show_play` and state, in plain words: the daily cap and
   that it is shared by every play on this LinkedIn account, the working hours, which
   rungs run alone and which wait for approval, and that the play acts from the
   human's own account. Then ask: "Launch?"
8. **Activate, only on a yes.** Call `gtm_update_play` with `playId` and
   `status: "active"`. If the note says the play fell back to draft for lack of a
   seat, go back to step 4.
9. **Explain the next 48 hours.** Day 1: sourcing and warm-up (visits, likes). Around
   day 2: first invites. First drafted messages wait for approval, visible with
   `gtm_list_approvals` here or in the cockpit at app.gtmotto.com. `gtm_play_stats`
   shows "warming up" instead of zeros until then. Offer the `weekly-play-review`
   skill for the first check-in.

## What to show the human

- After step 2: one line, the count and what you would change.
- After step 6: one table per lane, at most 10 rows, then one question.
- After step 7: a five-line summary (cap, hours, autonomy, account, what launching
  spends), then the question.
- After step 9: the 48-hour timeline and where approvals live.

## Edge cases

- `gtm_estimate_icp` returns null: no data provider on this deployment. Say so and
  continue; the preview in step 6 is the real test.
- `gtm_preview_source` says `skipped: no_seat`: step 4 is not done.
- The human names companies and no ICP: create with `account_list` on, leave
  `people_search` off, and use `gtm_update_icp` later to give the fit gate something
  to judge on.
- The human wants to try a lane for real before launching: `gtm_run_source` works on
  a draft play. Only on an explicit ask; it inserts leads and spends quota.
