---
name: setup
description: >
  Check the Atria plugin connection, find duplicate Atria servers, and run a read-only smoke test. Also compares the installed plugin version with what the server expects. Use when the user runs /atria:setup, asks whether Atria is connected, or sees Atria tools listed more than once.
disable-model-invocation: true
allowed-tools:
  - mcp__plugin_atria_atria__list_ad_accounts
  - mcp__plugin_atria_atria__list_ad_boards
  - mcp__plugin_atria_atria__list_followed_advertisers
  - mcp__plugin_atria_atria__list_notes
  - mcp__plugin_atria_atria__list_owned_brands
  - mcp__plugin_atria_atria__load_skill
  - Bash(claude mcp list*)
  - Bash(codex mcp list*)
---
# Atria setup check

Read-only. This skill never writes to the workspace and never calls a tool
that spends credits. Plugin version: **1.0.0**.

## 1. Is the server connected?

The plugin declares one MCP server named `atria` at
`https://api.tryatria.com/mcp`. It authenticates with OAuth: the first use
opens a browser sign-in.

- If Atria tools are not callable yet, tell the user to run `/mcp`, pick the
  `atria` server (it appears as `plugin:atria:atria` in Claude Code), and
  complete the sign-in. Do not try to work around it.
- If the tools are callable, continue.

## 2. Are there duplicate Atria servers?

Users who connected Atria before installing this plugin often have a second
or third copy of the same server. Each copy exposes the same tools, which
bloats the context and makes permission prompts confusing.

Run `claude mcp list` (or `codex mcp list` under Codex) and look for any other
server whose URL starts with `https://api.tryatria.com/mcp`. Common cases:

| What you see | Where it came from | What to suggest |
| --- | --- | --- |
| `atria` (user or project scope) | `claude mcp add --transport http atria https://api.tryatria.com/mcp` | `claude mcp remove atria -s user` (or `-s project`) |
| `claude.ai Atria` | The Atria connector added on claude.ai, synced into Claude Code | Disable it for this project from `/mcp`, or keep it and uninstall the plugin |
| `plugin:atria:atria` | This plugin | Keep |

Report what you found in one short table. Do not remove anything yourself;
the user decides which copy stays.

## 3. Smoke test

Call these in order and stop at the first failure:

1. `list_notes` - the notes this user keeps in Atria. Read the titles.
   If a note looks like standing context for this workspace (a brand
   direction, constraints, preferences), mention it; the user will not repeat
   it.
2. `list_ad_accounts` - connected ad accounts with platform and id.
3. `list_owned_brands` - the workspace's own brands.
4. `list_ad_boards` - shared boards.
5. `list_followed_advertisers` with `window=last_30d` - tracked competitors
   and how much of the tracking allowance is used.
6. `load_skill` with no arguments, if the tool is in the tool list - the
   skills this workspace has enabled in Atria's Skill Hub. If the response
   carries a minimum or latest plugin version, compare it with
   1.0.0 and, when this plugin is behind, print the update
   command for the current client (`/plugin marketplace update atria` in
   Claude Code, `codex plugin marketplace upgrade atria` in Codex). If the
   tool is not in the list, the server has not opened the Skill Hub yet; say
   so and move on.

## 4. Report

One compact summary:

- Connection status and any duplicate servers found.
- Counts: ad accounts (by platform), brands, boards, followed advertisers,
  notes, hub skills.
- Which skills fit what the workspace has. Bundled in this plugin:
  `account-weekly-report`, `competitor-teardown`, `setup`, `skill-hub`, `swipe-file`. Everything else Atria publishes is loaded through
  `/atria:skill-hub`: a workspace with ad accounts also wants
  creative-breakdown; one with filed competitors wants competitive-landscape;
  one with a brand profile wants creative-brief and ad-writer; an empty brand
  profile wants brand-onboarding.

If any call failed, quote the error and say which of the three layers it
points at: sign-in (401), permissions or plan (403), or a wrong id (an empty
result rather than an error).
