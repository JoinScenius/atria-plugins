---
name: competitor-teardown
description: >
  Take apart what one advertiser is running right now, from Atria's ad library. Live creatives, how long each has run, launch cadence, formats, hooks and CTAs, landing pages, a reach and durability read, and the evidence filed on an ad board with the conclusion written back. Use when the user names one competitor or brand and asks what they are advertising, wants that advertiser torn down, or asks what it launched over a period. For several competitors read together load competitive-landscape.
allowed-tools:
  - mcp__plugin_atria_atria__get_advertiser
  - mcp__plugin_atria_atria__get_followed_advertiser_stats
  - mcp__plugin_atria_atria__get_library_ad_creative_tags
  - mcp__plugin_atria_atria__get_library_ad_transcript
  - mcp__plugin_atria_atria__list_ad_boards
  - mcp__plugin_atria_atria__list_library_taggings
  - mcp__plugin_atria_atria__list_notes
  - mcp__plugin_atria_atria__resolve_advertiser
  - mcp__plugin_atria_atria__search_library_ads
---

# Competitor teardown

Everything here comes from the **ad library**: other advertisers' creative as
seen from outside. There is no spend or conversion data in it. Say so in the
report, and never mix these figures with the workspace's own ad-account
numbers.

## Ground rules

- Two bodies of data. The ad library is other advertisers' creative seen from outside, with no spend. The ad-account tools are this workspace's own spend and results. Say which one a figure came from; a conclusion from one does not transfer to the other.
- Start from an id the tools gave you. Advertiser ids and library ad ids are prefixed `m` or `t`; an account id is an Atria UUID; a brand id names one of the workspace's own brands and is never an advertiser id; a platform ad id is the platform's own, not a library ad id. A wrong id returns an empty result, not an error.
- Saving, following, and creating boards or brand entries change state teammates see; notes stay private to the user. Check what exists before creating anything, and confirm before writing.
- Transcription and image generation spend the workspace's AI credits. Read first, count what is needed, then ask.
- Call `list_notes` once per session before answering about this workspace. Write back what the user settles with `write_note`, never the data the tools can fetch again.

One advertiser, in depth. A set of competitors read together, share of voice,
saturation and whitespace against the workspace's own brand is
`competitive-landscape`; hand over when the user names more than one.


## 1. Resolve the advertiser

Call `resolve_advertiser` with whatever the user gave: brand name, domain,
landing-page URL, Facebook page link, Instagram profile.

- `match_type=exact` came from an identifier; use it.
- `match_type=fuzzy` came from a name. Several matches for one brand is normal
  (main page plus regional or product pages). Prefer the one with the most
  ads unless the user meant a specific market, and say which one you picked.
- `outcome=not_found` means the library does not track this advertiser yet.
  Offer `follow_advertiser` with a Meta Ads Library URL, but only after the
  user agrees: following is a workspace-level change and uses a tracking
  slot.

Then `get_advertiser` for the profile: lifetime ad count, whether the
workspace already follows it.

## 2. Pull what is live, not what exists

Use `search_library_ads` with `scope=advertiser` and the `advertiser_id`.
The default is what they are running right now: an advertiser's live set is
its current bet; its archive is everything it ever tried.

| Question | Arguments |
| --- | --- |
| What works for them (default) | `status=["active"]`, `order=most_active`, `collapse_variants=true` |
| What did they launch recently | `order=newest`, `launched_after=<date>` |
| What ran during a closed period | `active_since=<start>`, `launched_before=<end>` |
| What they have stopped | `status=["inactive"]`, `order=recently_ended` |
| Their biggest ads | `order=most_impressions` |

Always pass `fields` so pages stay cheap. A good default set:
`ad_id, advertiser_name, title, body_excerpt, cta_type, media_format,
display_format, start_date, end_date, days_running, preview_image_url,
platforms, themes, impression_rank, impression_trend, link_url, status`.

`page_size=20`. Read up to three pages (60 ads) unless the user asks for the
long tail. `collapse_variants=true` folds the same creative run twenty times
into one row.

## 3. Read the creative

- `get_library_ad_creative_tags` with up to 20 `ad_ids` per call gives hook,
  persona, angle, USP, desire, emotion. Send the ads worth understanding, not
  every row.
- If the tags come back with `untagged_reason=advertiser_not_followed`, tagging
  only runs for followed advertisers. Explain that `follow_advertiser` would
  start it (workspace-level, uses a slot) and ask before calling it.
- For the shape of the whole pool, `list_library_taggings` with
  `scope=advertiser`, one `tagging_type` per call (`hook`, then `persona`,
  then `ad_angle`), `window=last_90d`. Each group's `tagging_key` goes back
  into `search_library_ads` to see the ads behind it.
- If the workspace follows the advertiser, call
  `get_followed_advertiser_stats` **once** with `window=last_30d` (or the
  window the user asked about). It is expensive; do not re-run it with small
  changes. Read each count next to its own `based_on_ads`.
- Video scripts: `get_library_ad_transcript` is free and usually has the text
  already. `transcribe_library_ad` spends the workspace's AI credits and is not
  part of this skill's default flow. If a transcript is missing and the user
  wants it, say how many ads are involved and ask before transcribing.

## 4. Write the report

Deliver an HTML report the user can open and share - one self-contained file with inline CSS and no external assets: a header line naming subject, window and source; KPI cards; a gallery or table of the creatives with preview images linked to their assets; a timeline where dates matter; a notes section for the limits of the reading. Summarize in chat first and give the file path. If the user prefers chat, give the same sections as markdown.

Sections, in this order:

1. **Overview** — advertiser, lifetime ads, how many were pulled and with
   which ordering, the window covered, and the sentence "Source: Atria ad
   library (outside view, no spend data)".
2. **Longest-running creatives** — top 5 to 10 by `days_running` with
   preview, headline, CTA, format, platforms, start date. Label the cohort
   with the "Reach, momentum and durability" section below (heavyweight, slow burn,
   fading spike, breakout) from `impression_rank`, `impression_trend` and
   `days_running`, and print its caveats.
3. **Recent launches** — newest ads with launch dates; a launch timeline if
   there are more than ten.
4. **Format and placement split** — counts by `media_format` and `platforms`.
5. **Hooks, angles and CTAs** — from the creative tags and taggings; quote
   the `tagging_value` and give the count.
6. **Landing pages** — distinct `link_url` values with the number of ads
   pointing at each.
7. **What to take from it** — three to five observations tied to specific
   ads. Observations, not strategy: say what the advertiser does, not what the
   user should do, unless asked.

## 5. File the evidence and write the conclusion

Both steps close every teardown: the board is what the team can reopen, and the
note is what the user's next session starts from.

1. `list_ad_boards` first. Reuse a board that already covers this competitor;
   only when none does, propose a name and call `create_ad_board` after the
   user agrees. Nothing rejects a duplicate name.
2. `save_library_ad` for the 5 to 10 ads that carry the argument, not all of
   them. A board everyone scrolls past is the same as no board. Saving is
   workspace-wide.
3. `write_note` with what the teardown settled: the angles they own, the ones
   they leave alone, and which of those is worth testing. Name the board so
   the next session can reopen it. Do not copy the ad data into the note; the
   search tools remain the source of truth.

Downstream: a pattern worth answering goes to the skill named creative-brief
as a finding; a hook worth matching goes to the skill named ad-writer as a
reference; several competitors together go to competitive-landscape.

## Reach, momentum and durability

A relative read of a set of library ads on three signals, ending in a pattern
label. It reads volume and fatigue, never profit: a high-scoring ad may have
broad, cheap reach and poor economics.

Signals the library gives: **reach** is `impression_rank` within the
advertiser's own pool (1 is its top ad; never compare ranks across
advertisers; a missing rank means unmeasured, which is every TikTok ad and
anything below the per-advertiser cutoff) plus `reach` where disclosed;
**momentum** is `impression_trend` (rising, stable, falling), or reach over
`reach_period=7d` where disclosed; **durability** is `days_running` read with
`status`.

Rank each signal within the cohort into thirds (High, Mid, Low) and read the
pattern:

| Reach / Momentum / Durability | Pattern | Next step |
|---|---|---|
| High / High / Low | Breakout | study the hook and angle now; the ad to break down first |
| High / High / High | Heavyweight | their benchmark creative; the one they protect |
| High / Mid / High | Slow burn | steady; mine the appeal |
| Mid / Mid / High | Legacy plateau | long-running on flat momentum; a refresh candidate on their side |
| Low / Low / High | Dormant | running past anyone watching; ignore |
| High / Low / Low | Fading spike | reached big, momentum gone; a spent creative, not a current bet |
| Low / Low / Low | Unproven | too new or too small to read |

Reading durability: long and active is a durable bet, the strongest signal
the library gives and the easiest to over-read. It proves the advertiser's
economics, not yours, and a large advertiser with automated buying can leave
creative running past the point anybody is watching. Short and inactive was a
test they abandoned. Count distinct angles that survive, not distinct ads;
`collapse_variants` shows that. Recently launched advertisers have nothing
old, so for "who is new and moving" use rank and reach-change filters instead.

Print the caveats with any classification: a reach lens, not profitability;
percentiles within the named cohort only; which signals were unavailable.
