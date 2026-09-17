---
name: heyreach-migration
description: Move from HeyReach to GTM Otto using an exported HeyReach lead list CSV the user pastes or attaches. Extract the companies into a GTM Otto account_list lane, turn the list's titles, seniority, locations and industries into ICP facets, and explain honestly what does not carry over (sequences, by design; individual people, not yet). Use when the user mentions HeyReach, a lead list export, or migrating LinkedIn campaigns.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# HeyReach migration

HeyReach runs on lists of people and sequences of messages. GTM Otto runs on an ICP
and sourcing lanes, and writes each message for approval. So a migration is not a
copy: it keeps the companies and the targeting, and drops the sequences on purpose.
Say this in the first sentence, before touching a tool.

## Stop rules

- Never promise that a list of people will be imported. GTM Otto takes companies
  (`account_list`, max 100) and post URLs, not individual LinkedIn profiles.
- Never claim sequences carry over. They do not, by design.
- Do not switch a lane on or launch the play until the human has seen a preview and
  said yes. Never `gtm_run_source` unless asked.
- Do not paste the CSV back to the human; work from it and show summaries.

## Steps

1. **Get the export.** Ask for the HeyReach lead list export (CSV) pasted or
   attached. Typical columns: first name, last name, LinkedIn URL, job title, company
   name, company URL, location, industry, headcount, campaign, status. Column names
   vary; map them, do not assume.
2. **Say what carries and what does not.** Three lines:
   - Companies: yes, into an account list (up to 100 per play).
   - Targeting: yes, into the play's ICP (titles, seniority, locations, industries,
     sizes).
   - Sequences and the individual people: no. GTM Otto finds the buyers inside each
     company from the ICP and writes every message for approval; there is no
     sequence to import, and a people list is not supported yet.
3. **Extract companies.** Take the company name, or the company LinkedIn URL when
   present (it resolves without ambiguity). Dedupe case-insensitively. Drop rows
   whose status is "not interested", "replied negative", or blocked, unless the
   human says otherwise. If more than 100, rank by the human's priority (replied,
   connected, then pending) and tell them how many were cut.
4. **Derive ICP facets.** From the job title column, cluster into `titles` (the
   five to ten most common, normalised) and `seniority` (C-level, VP, Director,
   Manager, as they appear). From location and industry columns, `locations` and
   `industries`. From headcount, `sizes` as bands (`"11-50"`, `"51-200"`). Show the
   facets and ask the human to correct them before writing anything.
5. **Pick or create the play.** `gtm_list_plays`. If none fits, hand over to
   `launch-a-play` steps 1 to 3, with `account_list` on and `people_search` on, and
   come back with the `playId`.
6. **Write the ICP.** `gtm_update_icp` with `playId` and the facets from step 4.
   Only the facets you name change. Add `exclude` if the export shows clients or
   competitors the human never wants contacted.
7. **Read the reference once.** `gtm_sources_reference`.
8. **Load the companies.** `gtm_configure_source` with `playId`,
   `key: "account_list"`, `config: { companies: [...], fitGate: "standard" }`,
   `listMode: "append"`, no `on`. Report `truncated`.
9. **Check resolutions.** `gtm_show_sources`; list the `accounts[]` with
   `found: false` and ask for their LinkedIn page URLs; re-append those lines.
10. **Preview both lanes.** `gtm_preview_source` for `account_list` and for
    `people_search`. Show each sample (name, title, company, `alreadyALead`) and the
    `unmapped` facets. Tune with `gtm_update_icp` and preview again.
11. **Ask before switching on.** On yes, `gtm_set_source` with `on: true` for each
    lane. If the play is a draft, hand over to `launch-a-play` step 7 for the seat,
    the summary and the launch question.

## What to show the human

- After step 3: rows read, companies found, duplicates removed, rows dropped by
  status, and the first 10 companies.
- After step 4: the facets as a short list, with counts, and a request to edit.
- After step 10: one table per lane and one question.

## Honest answers to the questions they will ask

- "Can you import my 400 leads?" No. The 400 leads' companies, up to 100 per play,
  yes. The buyers inside them are found from the ICP, which is often better than the
  list, and sometimes the same people.
- "Where do my sequences go?" Nowhere. GTM Otto warms each person (visit, like,
  comment), invites them, and drafts a first message and follow-ups in the play's
  voice; each one waits for approval. Put the intent of the old sequence into the
  play's `goal` and `voice` with `gtm_update_play`.
- "What about my other campaigns?" One play per audience. Run this skill once per
  list, or merge lists that share an ICP.
- "Seats?" One LinkedIn account per play, the human's own, connected with
  `gtm_connect_linkedin`. No seat rotation.
