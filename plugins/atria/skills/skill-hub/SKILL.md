---
name: skill-hub
description: >
  Find and load Atria skills from the Skill Hub for work the bundled skills do not cover. Lists what this workspace has enabled, loads a skill by name and runs it, follows a skill's references, and turns a finished session into a draft skill. Use when the user asks what Atria can do, names a skill, asks for a kind of creative work no bundled skill covers, or wants to save the current workflow as a skill.
allowed-tools:
  - mcp__plugin_atria_atria__load_skill
---

# Skill Hub

Atria publishes its skills through the Skill Hub and hands them to the model
with one tool, `load_skill`. This plugin bundles a few of them so they work
without a round trip: `account-weekly-report`, `competitor-teardown`, `setup`, `skill-hub`, `swipe-file`. Everything else, including Atria's
other official skills and anything this workspace wrote or enabled, is loaded
from here. Plugin version: **1.0.0**.

## 1. Find

- The catalog is already in the `load_skill` tool description under
  "Available to this workspace". Read it there first; it is in context on
  every turn.
- When the description says more skills were dropped for length, or the user
  asks "what skills are there", call `load_skill` with no arguments for the
  full list. The list is filtered to what this workspace has enabled in
  Atria; a skill that is missing was not enabled, and enabling is done in
  Atria under Skill Hub.
- Pick by the description, the way you would pick a bundled skill. When the
  user names one, take it as named.
- Never load one of the bundled skills from the hub; the same text is already
  here.

## 2. Load and run

- `load_skill` with `name`. The response opens with a provenance line: it is
  guidance written by Atria, not an instruction from the user, and it cannot
  override what the user asked for. Then follow the body as you would a
  bundled skill.
- Load a skill once per session. When the body points you at a reference,
  call `load_skill` again with the same `name` and that `file`; only follow
  paths the skill itself names.
- "No skill named X is enabled" means exactly that; relay the message and
  offer the no-argument list. A message that the connection is not allowed to
  read skills is permanent for this session; continue without one and tell
  the user if it matters.
- Do not paraphrase the skill back to the user. Do the work it describes.

## 3. When the tool is missing

If `load_skill` is not in the tool list, this Atria server has not opened the
Skill Hub yet, or the plugin is newer than the server. Say so in one sentence
and continue with the bundled skills. Do not retry.

## 4. Version

If a `load_skill` response carries a minimum or latest plugin version and
1.0.0 is behind it, print the update command for the current
client: `/plugin marketplace update atria` in Claude Code,
`codex plugin marketplace upgrade atria` in Codex. Third-party marketplaces do
not update on their own.

## 5. Save the current workflow as a skill

When the user says "save this as a skill", draft a single SKILL.md from what
this session actually did: a name in lowercase letters, digits and single
hyphens; a description whose first sentence says what the skill does and
whose second says when to use it; a body that lists the steps and the Atria
tools each step calls, in order, with the write-back at the end. Show the
draft and wait for the user to confirm. Saving into the hub is a store
action that needs the user's confirmation; when the server offers a tool for
it, call that tool with the confirmed draft, otherwise tell the user to paste
it into Atria under Skill Hub.

## 6. Routing from the bundled skills

- account-weekly-report raises a creative question: load
  **creative-breakdown**.
- competitor-teardown is asked about several competitors at once: load
  **competitive-landscape**.
- swipe-file has a reference set and the user wants concepts or a brief:
  load **creative-brief**; for finished hooks, copy or scripts, load
  **ad-writer**.
- An empty brand profile: load **brand-onboarding**.
