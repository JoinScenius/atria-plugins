---
name: account-weekly-report
description: >
  Weekly performance report for one of the workspace's own connected ad accounts. Spend, results and cost per conversion on the account's own event against the week before, top ads by spend and by efficiency, creative tags at a glance, and the read written back to Atria. Monthly or any custom window on request. Use when the user asks how their account performed, wants a weekly or monthly report, asks which ads are spending, or wants two periods compared. For a qualitative read of the top creatives load creative-breakdown.
allowed-tools:
  - mcp__plugin_atria_atria__get_ad_account_ad
  - mcp__plugin_atria_atria__get_ad_account_summary
  - mcp__plugin_atria_atria__list_ad_account_ads
  - mcp__plugin_atria_atria__list_ad_account_creative_tags
  - mcp__plugin_atria_atria__list_ad_account_metrics
  - mcp__plugin_atria_atria__list_ad_accounts
  - mcp__plugin_atria_atria__list_notes
---

# Account weekly report

The numbers report, weekly by default. Everything here comes from the
workspace's **own ad accounts**: real spend and results. The job is to find
the one or two things that changed, not to describe the account; a dashboard
already shows every number. It is a different body of data from the
ad library and a conclusion from one does not transfer to the other. Label the
source in the report.

## Ground rules

- Two bodies of data. The ad library is other advertisers' creative seen from outside, with no spend. The ad-account tools are this workspace's own spend and results. Say which one a figure came from; a conclusion from one does not transfer to the other.
- Start from an id the tools gave you. Advertiser ids and library ad ids are prefixed `m` or `t`; an account id is an Atria UUID; a brand id names one of the workspace's own brands and is never an advertiser id; a platform ad id is the platform's own, not a library ad id. A wrong id returns an empty result, not an error.
- Saving, following, and creating boards or brand entries change state teammates see; notes stay private to the user. Check what exists before creating anything, and confirm before writing.
- Transcription and image generation spend the workspace's AI credits. Read first, count what is needed, then ask.
- Call `list_notes` once per session before answering about this workspace. Write back what the user settles with `write_note`, never the data the tools can fetch again.

What this skill does not do: read the creatives one by one. That is
`creative-breakdown`, and this report ends by pointing there when the numbers
raise a creative question.


## 1. Account, event, target

1. `list_notes` first: the user may have recorded the account's target, its
   conversion event, or how they want reports shaped.
2. `list_ad_accounts`. One account: use it. Several: ask, unless the user
   named one. Note the platform: TikTok accounts have no creative tags and
   rank on fewer metrics. `account_id` is the Atria UUID from this call.
3. `list_ad_account_metrics` once, `kind=standard`; again with
   `kind=facebook_custom_conversion` or `kind=atria_custom_metric` when the
   account's goal event is not in the standard set. Use `id` everywhere;
   display names are editable and collide.
4. **Anchor the report on cost per conversion for the account's own event**
   and, when the user has one, its target. Ask which event when the catalog
   holds several plausible ones and nothing in the conversation or notes says.
   Do not compute or mention ROAS unless the user asks for it or the goal is a
   return figure.

## 2. Windows

`period` accepts `yesterday`, `last_7d`, `last_14d`, `last_30d`, or `custom`
with `date_start` and `date_stop` (UTC, inclusive, at most 92 days, not in the
future). Presets end yesterday and are still settling; say so when the window
ends yesterday.

- **Weekly report:** the most recent **complete Monday-to-Sunday week** as a
  custom window, and the week before it as the comparison. Compute the dates
  from today's date; the presets do not align to weeks.
- **Monthly report:** the last complete calendar month and the month before.
- **Ad hoc:** the user's window and an equal-length prior window.

Change is reported as both the absolute difference and the percentage:
`(this - prior) / prior`.

## 3. Read the numbers

1. `get_ad_account_summary` for the window and again for the comparison
   window. Add `metrics` (comma-separated ids, up to 20) for the goal event
   and any custom metrics.
2. **Sanity check before reading a drop:** when the goal event falls sharply,
   pull the account's other conversion events for the same window; a
   tracking gap moves one event, a real drop moves them together. Say which
   it looks like.
3. `list_ad_account_ads` with `sort_by=spend`, `limit=10`: where the money
   went. Ranking covers the whole window.
4. `list_ad_account_ads` sorted on the goal event's cost metric
   (`sort_order=asc`) or the event count (`desc`): efficient ads that are not
   the biggest spenders. Note each one's spend so a tiny ad is not read as a
   winner.
5. `get_ad_account_ad` with `platform_ad_id` for the two or three ads worth
   showing: full copy, assets, campaign and ad-set names.
6. **Trend, on request only:** 4 to 8 equal windows via `period=custom`, one
   `get_ad_account_summary` each, for a spend and cost-per-conversion line.
   Say how many calls it takes before running it.

## 4. Creative at a glance (Meta only)

`list_ad_account_creative_tags` with the matching `period` (`last_7d`,
`last_14d` or `last_30d`; say so when it differs from the report window) and
`category=theme` or `media_format`. Report the two or three tags with the
clearest spread and the `coverage.tagged_spend_share`. Buckets overlap; never
sum them. `usp` and `key_message` are near-unique per asset; skip them.

Stop there. Which creatives to iterate, and why, is `creative-breakdown`.

## 5. Write the report

Deliver an HTML report the user can open and share - one self-contained file with inline CSS and no external assets: a header line naming subject, window and source; KPI cards; a gallery or table of the creatives with preview images linked to their assets; a timeline where dates matter; a notes section for the limits of the reading. Summarize in chat first and give the file path. Sections:

1. **Header** - account, platform, window and comparison window, the goal
   event by its own name, the target if any, and "Source: connected ad
   account".
2. **KPI cards** - spend, conversions, cost per conversion, impressions, CTR,
   CPM, each with the change versus the prior window; the goal target shown
   against the actual.
3. **Read** - three to five sentences: is the account on or off target, what
   moved, whether the movement is scaling dilution (spend up, cost slightly
   up, volume up), deterioration (spend flat, cost up), audience saturation
   (frequency up, CTR down, CPM up, when frequency is in the catalog), or a
   tracking gap. Say which and why.
4. **Top ads by spend** - table with preview, name, campaign, spend,
   conversions, cost per conversion.
5. **Efficient ads** - the second ranking with spend beside each.
6. **Creative at a glance** - the tags with coverage share.
7. **Notes** - settling days, partial tag coverage, TikTok gaps, `null`
   metrics, a blended-platform caveat when Meta and TikTok were combined.

Use the account's own metric display names from `metric_names`.

## 6. Write back, then hand off

Every report ends with `write_note`: on or off target, what moved and the
cause you settled on, and any decision the user made (a new target, an ad
the media team paused, a report cadence). Numbers stay out of the note; they
can be re-read. Update the existing weekly note rather than adding a new one
each week.

A creative question ("why is the top spender fading", "what are the winners
doing differently") is the skill named creative-breakdown. In the plugin,
the skill-hub entry loads it; elsewhere, load it by name.

