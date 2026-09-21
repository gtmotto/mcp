---
name: hiring-signals
description: Turn "companies hiring for X" into a GTM Otto Job offers lane (job_offers) that watches job posts daily and reaches the buyer inside each hiring company, not the recruiter. Use when the user mentions job posts, open roles, hiring signals, headcount growth, or wants to prospect companies that are recruiting for a role that proves they have the problem.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# Hiring signals

A job posting is a budget line someone already approved, with the pain written in
the job description. The `job_offers` lane finds companies posting a role every
day, then the buyers inside them, matched against the play's ICP. The one thing to
get right: the role you search for is the signal, the ICP is who you contact.

## Stop rules

- Read `gtm_sources_reference` before the first `gtm_configure_source`.
- Configuring is not switching on. Save with `on` omitted, preview, then ask before
  passing `on: true`.
- Never call `gtm_run_source` unless the human asked for it in this conversation; it
  inserts leads and spends the day's budget.
- Never leave `keywords` empty. The lane falls back to the ICP's `titles`, which
  means searching for the buyer's own role: usually wrong, always noisy.

## Steps

1. **Pick the play.** `gtm_list_plays`; if the human did not name one, ask. If none
   exists, hand over to `launch-a-play` and come back with the `playId`.
2. **Pick the role that means "about to buy".** Ask, or propose from the product:

   | They are hiring | It usually means | So the buyer is |
   |---|---|---|
   | first SDR / BDR | starting outbound, no tooling yet | founder, Head of Sales |
   | 2nd to 5th SDR | outbound works, they scale and consolidate | Head of Sales, RevOps |
   | first Head of Sales | the founder is handing off sales | the founder, before the hire lands |
   | RevOps / Sales Ops | stack consolidation, data, attribution | VP Sales |
   | Growth / Demand gen | pipeline pressure from the board | CMO, founder |
   | Customer Success Manager | churn or expansion is the quarter's problem | VP CS, COO |
   | Security / compliance engineer | a deal is blocked on a questionnaire | CTO, CISO |

   Write the row for the human's product: "what role does a company post the week
   before they would buy this?"
3. **Point the ICP at the buyer.** `gtm_show_play` to read the current ICP. If its
   `titles` are the hired role rather than the buyer row, fix it with
   `gtm_update_icp`: `titles` and `seniority` from the "so the buyer is" column,
   `sizes` where the signal means what the table says (a 5,000-person company hiring
   an SDR is not the same signal as a 30-person one), `locations` as needed. Only the
   facets you name change.
4. **Save the lane without switching on.** `gtm_configure_source` with `playId`,
   `key: "job_offers"`, `config: { keywords, postedWithinDays, fitGate }`:
   - `keywords`: the role being hired, in the words a posting would use
     (`"SDR"`, `"Head of Sales"`, `"Business Developer"`). One role per lane.
   - `postedWithinDays: 30`. Under 30 days is a live budget; over 90 it is filled or
     frozen. `null` means any age; propose it only when the human asks for volume.
   - `fitGate: "standard"`. Firmographics matter here, so keep the default.
5. **Preview.** `gtm_preview_source` with `playId` and `key: "job_offers"`. Show the
   size, the 10-person sample (name, title, company, `alreadyALead`) and `unmapped`.
   Ask: "are these the buyers, or the people being hired?"
6. **Tune.** Recruiters and talent partners in the sample: add `recruiter`, `talent`,
   `people` to the ICP `exclude` with `gtm_update_icp`. Too few: widen `sizes` or
   `locations`, or set `postedWithinDays: 60`. Too many at big companies: narrow
   `sizes`. Save (merge: only the fields you send change) and preview again.
7. **Switch on, only on a yes.** Ask "switch the lane on?". On yes, `gtm_set_source`
   with `playId`, `key: "job_offers"`, `on: true`. Say that it runs on the next
   5-minute tick if the play is active and not paused, then once every 24 hours, up
   to 25 new people per run; on a draft play it waits for launch.
8. **Set the opener.** The note and first message should reference the hire, not the
   product: "saw you're opening a first SDR role; the first 90 days usually go into
   list building". Put that in the play's `voice` or `goal` with `gtm_update_play`
   if the human wants it. The drafts wait for approval either way.

## What to show the human

- The row you chose (signal role, what it means, who to contact) before saving.
- One table per preview, at most 10 rows, plus the size and its precision.
- After switching on: "runs on the next tick, then daily; one buyer per company per
  signal; I will not run it by hand unless you ask".

## Edge cases

- The human names the buyer and the hired role as the same title ("we sell to Heads
  of Sales, find companies hiring a Head of Sales"): that works, but say that the
  person found is the one about to leave the role or the one about to be replaced,
  and propose the founder as the buyer instead.
- `blocked: icp_empty` in `gtm_show_sources`: step 3 is not done. The lane needs
  `titles` to know who inside the company to take.
- Reposts every two weeks are their own signal (they are struggling to hire). The
  lane dedupes companies it already added, so a repost does not double-contact.
