---
name: swipe-file
description: >
  Build a swipe file of reference ads from Atria's ad library and file it on a shared board. Three modes: search by theme, angle, format, industry or market; find library lookalikes of the workspace's own winning ads; or review what a board already holds. Use when the user wants inspiration, examples of a kind of ad, ads about a topic or using a hook, ads like their best performers, a reference set for a brief, or a read of one of their boards.
allowed-tools:
  - mcp__plugin_atria_atria__get_ad_account_ad
  - mcp__plugin_atria_atria__get_ad_account_creative_tags
  - mcp__plugin_atria_atria__get_ad_board
  - mcp__plugin_atria_atria__get_library_ad_creative_tags
  - mcp__plugin_atria_atria__list_ad_account_ads
  - mcp__plugin_atria_atria__list_ad_accounts
  - mcp__plugin_atria_atria__list_ad_boards
  - mcp__plugin_atria_atria__list_library_taggings
  - mcp__plugin_atria_atria__list_notes
  - mcp__plugin_atria_atria__search_ad_templates
  - mcp__plugin_atria_atria__search_library_ads
---
# Swipe file

A swipe file is the team's shared set of reference ads, filed on Atria's ad
boards. This skill builds one three ways, all from **paid ads** in Atria's ad
library (never organic posts):

- **Mode A - search the library** (steps 1 to 5): by theme, angle, format,
  industry, market or audience.
- **Mode B - lookalikes of your own winners** (step 2b, then 3 to 5): read
  what the workspace's top spenders share and find library ads built the same
  way.
- **Mode C - review a board** (the "Board mode" section): what the team has
  already collected, what runs longest, what has stopped.

If a request is ambiguous ("UGC", "testimonial" can mean paid or organic),
say you are searching paid ads and continue; ask only when the user clearly
means organic content. The complete argument guide for the search is the
"Search filters" section at the end; read it before building a query.

## Ground rules

- Two bodies of data. The ad library is other advertisers' creative seen from outside, with no spend. The ad-account tools are this workspace's own spend and results. Say which one a figure came from; a conclusion from one does not transfer to the other.
- Start from an id the tools gave you. Advertiser ids and library ad ids are prefixed `m` or `t`; an account id is an Atria UUID; a brand id names one of the workspace's own brands and is never an advertiser id; a platform ad id is the platform's own, not a library ad id. A wrong id returns an empty result, not an error.
- Saving, following, and creating boards or brand entries change state teammates see; notes stay private to the user. Check what exists before creating anything, and confirm before writing.
- Transcription and image generation spend the workspace's AI credits. Read first, count what is needed, then ask.
- Call `list_notes` once per session before answering about this workspace. Write back what the user settles with `write_note`, never the data the tools can fetch again.

## 1. Settle the scope in one exchange

Collect what the user already said; ask for the rest only if the search
cannot be built without it:

- Topic or product category.
- Message angle → the `theme` enum (`testimonial`, `before_after`,
  `discount`, `problem_solution`, `UGC`, ...).
- Format → `media_format` (`image`, `video`, `carousel`).
- Market and language → `target_countries` or `main_country`, `language`.
- Freshness or durability → `launched_after`, or `min_days_running` with
  `order=most_active`.

## 2. Build the search

Start broad, then narrow. Typical first call:

```
search_library_ads
  scope=library
  theme=[...]            # message angle
  media_format=[...]
  target_countries=[...]
  order=most_active       # proven creatives first
  collapse_variants=true  # one row per distinct creative
  page_size=20
  fields=[ad_id, advertiser_name, title, body_excerpt, cta_type,
          media_format, days_running, start_date, preview_image_url,
          themes, platforms, impression_rank, link_url]
```

Rules that save wasted calls:

- `query` searches ad copy and advertiser name. It is for topics ("Black
  Friday", "protein"), not for hooks or audiences.
- Hooks, personas and angles live in tags. Get a `tagging_key` from
  `list_library_taggings` (`scope=industry` for a category sweep, or
  `scope=advertiser` from a seed advertiser) or from
  `get_library_ad_creative_tags`, then pass `tagging_type` plus
  `tagging_key`. Never hand-write a key.
- For "proven" ads, `max_impression_rank=5` keeps each advertiser's top five;
  `max_impression_percentile=0.1` is the relative version.
- When one advertiser floods the results, re-run with
  `exclude_advertiser_ids`.
- Two or three refinements is normal. Stop when the page is mostly relevant.

`search_ad_templates` is a different set: Atria's hand-picked static
references, tagged with their own `theme` vocabulary
(`discount_promotion`, `ugc_testimonial`, `product_demo`, ...). Use it when
the user asks how a kind of static ad is built. Its theme keys are not the
library's, and a sparse result means the set has nothing like that, not that
no such ad exists.

## 2b. Mode B: lookalikes of your own winners

When the user wants library ads like their own best performers:

1. `list_ad_accounts`, then `list_ad_account_ads` with `sort_by=spend` (or
   the goal event's cost metric) for the top 5 to 10 winners.
2. `get_ad_account_ad` for each: assets and `video_id` / `image_hash`; then
   `get_ad_account_creative_tags` for their theme, visual hook, persona,
   core desire and offer type. Untagged winners (`not_tagged_yet`) are read
   from their copy and preview instead.
3. Pick the two or three signals the winners share (a hook type, a persona, a
   theme). Those are the query, not the products or the brand.
4. Translate them into library terms: `theme` enum values directly;
   hooks, personas and angles through `list_library_taggings`
   (`scope=industry` for the brand's industry) to find the matching
   `tagging_key`, then `search_library_ads` with `tagging_type` and
   `tagging_key`, `collapse_variants=true`, `order=most_active`.
5. Exclude the workspace's own advertiser id with `exclude_advertiser_ids` if
   the brand is in the library.

Say clearly which body of data each half came from: the winners are the
workspace's own account; the lookalikes are other advertisers' creative.

## 3. Read and group

Pull creative tags for the ads worth keeping (`get_library_ad_creative_tags`,
up to 20 ids per call). Group by what the user asked for: theme, hook,
format, or advertiser. For each group give two or three examples with
preview, advertiser, headline, days running.

## 4. Present

Chat: the groups, each with its examples and a one-line "why it is here".
When the user wants something to share, deliver an HTML gallery the user can
open and share - one self-contained file with inline CSS and no external assets:
a header line naming the topic, the window and the source; the groups from step
3 as a gallery with preview images linked to their assets; a notes section for
the limits of the reading and the filters used, so the search is reproducible.
Summarize in chat first and give the file path.

## 5. File the keepers

Saving is a workspace act: teammates see the board and the saved ads.

1. `list_ad_boards` first. Reuse a board whose `path` fits.
2. If none fits, propose a board name and create it only after the user
   agrees (`create_ad_board` is not in this skill's allowed list on purpose;
   the user approves that call).
3. Confirm the list of ads to save, then `save_library_ad` for each with
   `ad_board_ids` and optional `tags`. Saving is additive and safe to repeat.

If the user settles on a direction while researching, record it with
`write_note`; the ads themselves stay in the board, not in the note.

Downstream: a reference set becomes concepts and a brief in
`creative-brief`, and finished lines in `ad-writer`.

---

## Board mode

Boards hold **library ads** the team saved: other advertisers' creative. They
never contain the workspace's own uploaded assets.

### 1. Find the board

`list_ad_boards` returns the whole tree flat, one row per board, with `path`
already assembled (`Q3 launch / Hooks / UGC`) and `ads_filed_here` counting
ads filed directly in that board. Match the user's name against `path`; when
two boards could match, show both paths and ask.

`get_ad_board` adds the direct children. A parent's count does not include
its children, so a review of a parent means reviewing each child too. Say
which boards the review covers.

### 2. Read the ads

`search_library_ads` with `scope=ad_board` and `ad_board_id`. Page through
with `cursor` until `page.cursor` is empty (`page_size=20`). Ask for the
fields the review needs:
`ad_id, advertiser_name, title, body_excerpt, cta_type, media_format,
display_format, start_date, end_date, status, days_running,
preview_image_url, themes, platforms, link_url, saved_details`.

`saved_details` carries the workspace's own tags and filing, which is what
makes a board review different from a library search. To review everything
the workspace saved regardless of board, use `scope=saved` with
`order=saved_newest`.

Then `get_library_ad_creative_tags` in batches of 20 for hooks, personas and
angles. Ads from advertisers nobody follows come back untagged
(`advertiser_not_followed`); report them as untagged rather than as having
no angle.

### 3. Analyze

Answer these, each with counts and two or three example ads:

- **Themes and hooks** — what the team keeps saving, by `themes` and by the
  `hook` tag.
- **Advertisers** — who the board leans on; flag when one advertiser is more
  than a third of it.
- **Formats and placements** — `media_format`, `display_format`, `platforms`.
- **Durability** — longest-running ads by `days_running`; ads with
  `status=inactive` that have stopped since they were saved.
- **Landing pages** — distinct `link_url` values.
- **Filing** — workspace tags in `saved_details`; ads saved with no tag.

Keep observations tied to specific ads; a board review is evidence, not
strategy, unless the user asks for recommendations.

### 4. Present

Chat summary first. For a shareable version, deliver an HTML dashboard the
user can open and share - one self-contained file with inline CSS and no
external assets: a header line naming the board, the boards covered and the
source; KPI cards for the theme, advertiser and format split; a gallery of the
standouts with preview images linked to their assets; the stopped ads; a notes
section for the limits of the reading. Give the file path.

### 5. Tidying

Only on an explicit request. Removing an ad from a board
(`remove_library_ad_from_ad_board`) or unsaving it (`unsave_library_ad`) is
visible to the whole team and neither is in this skill's allowed list, so
the user approves each call. Propose the list, get a yes, then act.

## Search filters

`search_library_ads` finds every library ad; `scope` decides where it looks.

| `scope` | Needs | Searches |
|---|---|---|
| `library` (default) | - | everything |
| `advertiser` | `advertiser_id` | one advertiser's ads |
| `ad_board` | `ad_board_id` | one of the workspace's boards |
| `saved` | - | every ad the workspace saved; allows `order=saved_newest` |
| `followed_advertisers` | - | every advertiser the workspace tracks |

**Dates.** `launched_after` / `launched_before` bound when an ad first ran
("what did they launch last month"). `active_since` keeps ads still running
on or after a date, including ones launched long before ("what are they
running now"). A closed period is `active_since=<first day>` plus
`launched_before=<last day>`. There is no end-date filter; for stopped ads use
`status=["inactive"]` with `order=recently_ended`.

**Ordering.** `newest`, `oldest`, `most_active` (longest-running first),
`recently_ended`, `best_match` (needs `query`, cannot combine with
`collapse_variants`), `most_impressions`, `most_saved`.

**Cheap pages.** Always pass `fields` so rows carry exactly what the question
needs plus `ad_id`. `page_size` is 1-20; continue with `cursor`.
`collapse_variants=true` returns one row per distinct creative.

**Text versus tags.** `query` searches ad copy and advertiser name: topics,
products, events. Hooks, personas, angles, USPs, desires and emotions live in
tags: get a `tagging_key` from `list_library_taggings` or
`get_library_ad_creative_tags`, then pass `tagging_type` plus `tagging_key`.
Never hand-write a key; searching `query` for a hook returns noise.

**Creative.** `media_format` (image, video, carousel, other);
`display_format` (image, video, carousel, multi_images, multi_videos, dco,
dpa); `video_length` (0-15, 15-30, 30-60, 60-120, 120-); `theme` from the
library vocabulary: feature_callout, problem_solution, discount, showcase,
before_after, testimonial, social_proof, announcement, holiday_festival,
comparison, seasonal, question, scarcity, statistics, authority_celebrity,
interface, quiz_survey, quote, unboxing, 3_reasons, founder_story,
negative_marketing, media_press, behind_the_scenes, post_it_notes, meme,
case_study, us_vs_them, ebook, UGC, absurd_alternatives; `cta` by rendered
button label (shop_now, learn_more, sign_up, download, book_now, get_offer,
buy_now, subscribe, contact_us, no_button, others); `min_body_length` /
`max_body_length`; `min_creative_duplicates` / `max_creative_duplicates`
(`max_creative_duplicates=1` means never reused).

**Who and where.** `industry` (Apparel & Accessories, Appliances, Baby, Kids &
Maternity, Beauty & Personal Care, Book/Publishing, Business Services,
Charity, NFP & NGO, E-Commerce, Education, Event, Financial Services, Fitness,
Sports & Outdoors, Food & Beverage, Games, Government, Health & Medical, Home
Improvement & Garden, Life Services, Pets, Science, Technology & Engineering,
Supplements & Pharmaceuticals, Travel & Hospitality, Vehicle &
Transportation); `platform` (facebook, instagram, messenger,
audience_network, threads, whatsapp, tiktok, linkedin); `language` as ISO
639-1; `target_countries` (any delivery country), `main_country` (largest
disclosed reach), `exclude_target_countries`; `audience_gender`,
`min_audience_age` / `max_audience_age` from transparency disclosures (any
bound excludes ads without them); `exclude_advertiser_ids` for "who else does
this"; `partner_ads=true` for branded content; `tags` for the workspace's own
filing (rows then carry `saved_details`).

**Proven and rising.** `max_impression_rank=N` keeps each advertiser's top N
ads; `max_impression_percentile=0.1` is the relative version;
`impression_trend` rising / stable / falling; rank movement with
`min_impression_rank_delta` / `max_impression_rank_delta` and
`impression_rank_delta_period` (negative means climbed); reach with
`min_reach` / `max_reach` and `reach_period`, growth with
`min_reach_pct_change` and `reach_pct_change_period`. Rank, trend and page
likes are Meta signals and drop every TikTok ad.

**Taggings.** `list_library_taggings` groups a set of ads by one dimension:
`tagging_type` (hook, persona, ad_angle, usp, desire, emotion, theme,
creative, ad_copy, headline, landing_page) × `scope` (`advertiser`,
`advertiser_set`, `industry`) × `window` (up to `last_180d`); `order` by
`ads`, `share`, `longest_running` or `most_recent`. Each group's
`tagging_key` goes straight back into the search.

**Templates are a different set.** `search_ad_templates` is Atria's
hand-picked static references with its own `theme` vocabulary
(discount_promotion, before_after, ugc_testimonial, product_demo, lifestyle,
problem_solution, feature_highlight, social_proof, urgency_scarcity,
new_arrival_launch, seasonal_holiday, bundle_upsell, free_trial_sample,
how_to_tutorial, comparison, unboxing, founder_brand_story,
emotional_storytelling, educational_content, celebrity_kol,
minimalist_product, user_challenge_trend, retargeting_reminder). A library
theme key passed there is refused; a sparse result means the set holds
nothing like it, not that no such ad exists.
