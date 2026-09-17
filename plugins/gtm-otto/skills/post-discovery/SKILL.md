---
name: post-discovery
description: Turn "people complaining about X on LinkedIn" into a GTM Otto Post discovery lane (competitor_posts) with an intent, anchors, gates and a previewed sample. Use when the user wants to reach people who post or comment about a pain, a competitor, a tool or a chore, or pastes a LinkedIn post URL and asks to reach its engagers.
license: MIT
compatibility: Requires the GTM Otto MCP server (https://agent.gtmotto.com/mcp) connected in the client.
metadata:
  author: gtmotto
  version: "1.0"
---

# Post discovery

The `competitor_posts` lane ("Post discovery" in the cockpit) searches LinkedIn every
day for posts that match an intent, judges each post, and takes the people who
engaged with it. It is the lane for "people complaining about X": intent beats
firmographics here.

## Stop rules

- Read `gtm_sources_reference` before the first `gtm_configure_source`.
- Configuring is not switching on. Save with `on` omitted, preview, then ask before
  passing `on: true`.
- Never call `gtm_run_source` or `gtm_forget_harvested_posts` unless the human asked
  for it in this conversation; both cost.
- Do not configure the retired `post_engagers` lane. Named posts go in
  `competitor_posts.postUrls`.

## Steps

1. **Pick the play.** `gtm_list_plays`; if the human did not name one, ask. If none
   exists, hand over to `launch-a-play` and come back with the `playId`.
2. **Write the intent.** One or two sentences, in the operator's own words and
   language, describing who you are hunting. It does three jobs: the query writer
   writes LinkedIn searches from it, the post scorer judges each found post against
   it, the comment scorer measures each engager against it. Keep it about the
   person's situation, not about your product.
3. **Add anchors.** `competitors`: tools, brands, chores the posts would name
   (`"Sales Navigator"`, `"Waalaxy"`, `"prospects froids"`). Max 25. These are the
   fallback searches when the model cannot write queries; three to eight is plenty.
4. **Add named posts, if any.** `postUrls`: any LinkedIn post URL or
   `urn:li:activity:...`. Max 25. These are harvested as they are, without the post
   gate, because the human picked them.
5. **Set the gates.** Start moderate: `postGate: 70` (a found post must score 70 or
   more against the intent), `minReactions: 25` (skip posts nobody saw),
   `intentGate: null` (off until the sample shows noise), `fitGate: "loose"` (role
   and seniority only; intent counts more than firmographics on engager lanes).
   `commenters: true`, `reactors: true`. Note that `postGate: 0` is not off; `null`
   is off.
6. **Save without switching on.** `gtm_configure_source` with `playId`,
   `key: "competitor_posts"`, `config: { intent, competitors, postUrls, postGate,
   minReactions, intentGate, fitGate, commenters, reactors }`. Show `truncated` if
   anything was cut.
7. **Preview.** `gtm_preview_source` with `playId` and `key: "competitor_posts"`.
   Show the size, the 10-person sample (name, title, company, `alreadyALead`) and
   `unmapped`. Ask whether these are the right people.
8. **Tune.** Too broad: raise `postGate` to 80, switch `intentGate` to 70, or set
   `fitGate: "standard"`. Too narrow: lower `minReactions`, drop `postGate` to 60, add
   anchors. Save with `gtm_configure_source` (merge: only the fields you send
   change) and preview again.
9. **Switch on, only on a yes.** Ask "switch the lane on?". On yes,
   `gtm_set_source` with `playId`, `key: "competitor_posts"`, `on: true`. Say that it
   runs on the next 5-minute tick if the play is active and not paused, then once
   every 24 hours; on a draft play it waits for launch.

## Worked example

The human says, in French: "Je veux toucher les gens qui râlent de payer 100 balles
par mois pour Sales Navigator."

- `intent`: "Commerciaux et fondateurs francophones qui trouvent Sales Navigator
  trop cher pour ce qu'il apporte, se plaignent de payer environ 100 euros par mois,
  ou cherchent une alternative pour prospecter sur LinkedIn."
- `competitors`: `["Sales Navigator", "LinkedIn Sales Navigator", "prospection
  LinkedIn", "100 euros par mois", "Waalaxy", "Lemlist"]`
- `postUrls`: the post the human pasted, if any.
- Gates: `postGate: 70`, `minReactions: 25`, `intentGate: null`,
  `fitGate: "loose"`, `commenters: true`, `reactors: true`.

Preview. If the sample is full of vendors selling alternatives, set
`intentGate: 70` so each engager's comment is scored against the intent, and add
`"vendeurs d'outils de prospection"` to the play's ICP `exclude` with
`gtm_update_icp`. Preview again, then ask before switching on.

## What to show the human

- The intent and anchors as you wrote them, before saving. Let them edit.
- One table per preview, at most 10 rows, plus the size and its precision.
- After switching on: "runs on the next tick, then daily; I will not run it by hand
  unless you ask".

## Tuning trap

The lane remembers every post it already paid to harvest (`harvestedPosts` in
`gtm_show_sources`), so after a real run, changing a gate and running again answers
"no new posts". `gtm_forget_harvested_posts` clears that memory; the next run
re-fetches every forgotten post's engagers, paid. Offer it only after a deliberate
settings change the human wants re-tested, and only call it when they say yes.
