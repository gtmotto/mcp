---
name: instantly-to-linkedin
description: Add LinkedIn to an Instantly email campaign from the same chat. With the Instantly MCP and the GTM Otto MCP both connected, take the companies of leads who replied to or opened a campaign, dedupe them, and append them to a GTM Otto account_list lane so buyers there get warmed and invited on LinkedIn. Use when the user mentions Instantly, wants to follow up email engagement on LinkedIn, or asks to turn campaign replies into LinkedIn outreach.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) and the Instantly MCP both connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# Instantly to LinkedIn

Email found the interest; LinkedIn continues it. This skill moves the companies that
engaged with an Instantly campaign into a GTM Otto `account_list` lane. It never
touches Instantly's data and never sends anything.

## Stop rules

- Do not switch the lane on until the human has seen the preview and said yes.
- `listMode: "append"` always. Never replace an existing account list without
  asking; the human may have companies there you do not know about.
- Never call `gtm_run_source` or activate the play unless asked in this conversation.
- If the Instantly MCP is not connected, say so and offer the manual route: the human
  pastes company names or domains and you continue at step 4.

## Steps

1. **Check both sides.** `gtm_whoami` for the GTM Otto plan and `next`. Confirm the
   Instantly tools are present by listing campaigns with the Instantly MCP. If the
   human did not name a campaign, show the list (name, status, sent, replies) and ask.
2. **Pull the engaged leads.** With the Instantly MCP, fetch the campaign's leads and
   keep those whose status shows a reply, or an open if the human said "opened".
   Prefer replies: an open is a weak signal. Exclude leads marked out of office,
   bounced, unsubscribed, or "not interested" if Instantly labels it.
3. **Reduce to companies.** For each lead take the company name if present, else the
   domain of the email address. Drop free mail domains (gmail.com, outlook.com,
   yahoo.*, icloud.com, hotmail.*). Dedupe case-insensitively. Prefer the domain over
   the name when both exist: a domain resolves to one LinkedIn page, a name can be
   ambiguous.
4. **Respect the cap.** `account_list` takes 100 companies per play. If there are
   more, keep the ones with replies first, then the most recent opens, and tell the
   human how many were cut. `gtm_show_sources` on the target play shows how many are
   already in the list.
5. **Pick the play.** `gtm_list_plays`. If no play fits, hand over to
   `launch-a-play` with `account_list` as the first lane, then come back.
6. **Read the reference once.** `gtm_sources_reference`, if not already read this
   session.
7. **Append the companies.** `gtm_configure_source` with `playId`,
   `key: "account_list"`, `config: { companies: [...], fitGate: "standard" }`,
   `listMode: "append"`, and no `on`. Show `truncated` if any were cut.
8. **Check resolutions.** `gtm_show_sources` and read `accounts[]`. A line with
   `found: false` is a spelling or ambiguity problem: ask the human for the company's
   LinkedIn page URL and re-append that line.
9. **Preview.** `gtm_preview_source` with `playId`, `key: "account_list"`. Show the
   size, the 10-person sample (name, title, company, `alreadyALead`) and what each
   company resolved to. The ICP's `titles` and `seniority` decide who inside each
   company is a buyer; if the sample is the wrong function, fix it with
   `gtm_update_icp` and preview again.
10. **Ask before switching on.** "Switch the account list on? It runs on the next
    tick if the play is active, then daily, 10 companies per query, rotating." On
    yes, `gtm_set_source` with `on: true`. If the play is a draft, say that nothing
    runs until it is launched, and offer `launch-a-play` step 7 onward.

## What to show the human

- After step 3: the count of engaged leads, the count of unique companies, and the
  first 10 as a list. Ask before continuing if the human named the wrong campaign.
- After step 8: any `found: false` lines, as a short list with a request for URLs.
- After step 9: the sample table and one question.

## Why companies and not people

GTM Otto takes companies and post URLs, never a list of individual profiles. The
lead who replied by email is often not the buyer on LinkedIn; the account list finds
the right people inside each company from the play's ICP. If the human wants the exact
same people, say plainly that a people list is not supported yet.

## Edge cases

- Same domain under several campaigns: dedupe across campaigns before appending.
- Agencies with client domains in the lead list: ask whether the client companies or
  the agency are the target.
- A company already `alreadyALead` in the sample: fine, the lane will skip it.
