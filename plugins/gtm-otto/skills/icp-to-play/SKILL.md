---
name: icp-to-play
description: Turn a sentence, a website URL or a rough persona into the seven GTM Otto ICP facets, size the audience with gtm_estimate_icp, and write it to a play's people_search lane, on a new draft or an existing play. Use when the user describes who they sell to, asks to size an audience, wants to change a play's targeting, normalize job titles, or add exclusions, and the GTM Otto MCP is connected.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# ICP to play

Input: "CFOs at Swiss fintechs", a website URL, or a paragraph about the buyer.
Output: an ICP card in seven facets, a company count for the audience, and a
`people_search` lane that runs every day. GTM Otto compiles the facets into the
LinkedIn query itself and normalises odd labels ("Swiss" to Switzerland, "Fintech"
to Financial Services), so plain words are fine.

## Stop rules

- `gtm_estimate_icp` and `gtm_preview_source` write nothing. Use them freely.
- Never pass `activate: true` to `gtm_create_play`, and never call `gtm_update_play`
  with `status: "active"` from this skill. Hand over to `launch-a-play` for that.
- The first `gtm_update_icp` on a play forks its own ICP row; no other play's
  targeting changes. Say so if the human worries about it.
- Never call `gtm_run_source` unless the human asked for it in this conversation.

## Steps

1. **Extract the seven facets.** Only name the facets the human gave or that follow
   from the product; leave the rest out. An empty facet is "any", not "unknown".

   | Facet | What goes in | Example |
   |---|---|---|
   | `titles` | every wording of the role, current job only | CFO, Chief Financial Officer, VP Finance, Head of Finance |
   | `seniority` | owner, cxo, vp, director, manager, senior, entry | cxo, vp, director |
   | `languages` | the language the buyer posts in | French, English |
   | `industries` | LinkedIn industry names, plain words | Financial Services, Fintech, Banking |
   | `sizes` | headcount bands | 11-50, 51-200, 201-500 |
   | `locations` | countries, regions, metro areas | Switzerland, Geneva |
   | `exclude` | never contact, absolute at every level | fractional, freelance, interim, intern, student, recruiter, agency, consultant |

   Bands: `1-10`, `11-50`, `51-200`, `201-500`, `501-1000`, `1001-5000`,
   `5001-10000`, `10001+`. "SMB" is 11 to 200, "mid-market" 201 to 1000,
   "enterprise" 1001+. From a website URL: the pricing page gives the size band, the
   customer logos the industry, the docs or integrations page the buyer's function.
2. **Normalise titles.** A title is a family: the acronym and the long form, the VP,
   Head, Director and Lead variants where company size makes them the same person,
   the local-language forms for each `languages` entry, and the adjacent function
   that owns the same budget (RevOps for a sales tool). Always add to `exclude`:
   `fractional`, `freelance`, `interim`, `intern`, `assistant to`, `student`,
   `former`, `ex-`, `retired`, `looking for`. Add `recruiter`, `agency`,
   `consultant`, `advisor` unless those are the buyer.
3. **Size it.** `gtm_estimate_icp` with `industries`, `locations`, `sizes` only
   (titles do not change the company count). Show "about N companies match". A daily
   lane takes up to 25 new people per run, so a few thousand is plenty. Under 50:
   propose widening one facet. Over 5,000: propose adding an industry or narrowing
   the size band. Iterate until the human is satisfied with the number. A `null`
   estimate means no data provider on this deployment, not an error; continue, the
   preview in step 5 is the real test.
4. **Write it.** Either:
   - **Existing play**: `gtm_list_plays`, pick it, then `gtm_update_icp` with
     `playId` and the facets. Only the ones you name change.
   - **New play**: `gtm_create_play` with `name` (the audience in three words),
     `websiteUrl` or `productId` from `gtm_whoami`, `icp` with the facets, and
     `sources: [{ key: "people_search", on: true, config: { keywords, fitGate:
     "standard" } }]`. No `activate`: it stays a draft. Show the returned `playId`
     and the cockpit link.

   `keywords` on `people_search` are extra free-text terms on top of the ICP
   (`"outbound"`, `"HubSpot"`), for the chore or the tool, never for the title. Alone
   they are a complete search; the lane runs on an empty ICP if keywords are set.
5. **Preview.** `gtm_preview_source` with `playId` and `key: "people_search"`. Show
   the size with its precision, the 10-person sample (name, title, company,
   `alreadyALead`) and the `unmapped` facets: the ones LinkedIn cannot filter on (a
   language, "does their own prospecting") that the fit gate judges per person.
   `standard` needs one match, `strict` needs all, `loose` is role and seniority
   only; `exclude` is absolute at every level. Ask whether these are the right
   people; two or three rounds of `gtm_update_icp` and preview is normal.
6. **Hand off.** The play is a draft, or the existing play keeps its status. To
   launch, follow `launch-a-play`. Never activate from here.

## What to show the human

- The ICP card as seven short lines, before any call. Let them edit.
- After step 3: one line, the count and what you would change.
- One table per preview, at most 10 rows, plus the size and the unmapped facets.
- After step 4 on a new play: the `playId`, the cockpit link, and "it is a draft".

## Edge cases

- The human gives a Sales Navigator boolean instead of facets: split it back into
  `titles` (the OR family) and `exclude` (the NOT group). Keywords go to
  `keywords`, not `titles`.
- `blocked: icp_empty` on `people_search` in `gtm_show_sources`: step 4 was skipped
  or wrote nothing. Fill `titles` at least.
- The human names companies rather than a persona: that is an `account_list` lane;
  keep `people_search` off and still write the ICP so the fit gate has something to
  judge on. See `launch-a-play`.
