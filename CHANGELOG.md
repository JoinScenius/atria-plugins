# Changelog

All notable changes to the Atria plugin are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow semver.

## [1.2.4] - 2026-11-12

### Fixed

- Remove bundled skills' MCP and shell tool pre-approvals to meet the Claude
  directory policy for organizations without Verified status. Tool calls now
  follow the user's existing permission settings; workflow instructions remain.
- Remove the unsupported `oauth_resource` field from the shared Atria MCP
  connection configuration. The HTTPS endpoint and OAuth sign-in remain.

## [1.2.3] - 2026-11-12

### Changed

- Update the Claude Atria plugin and marketplace descriptions to cover image and
  video creation alongside advertising research and ad-account analysis.
- Expand the README with Raya's project page, direct project editing, example
  requests, and workspace-credit guidance. Clarify the focused Ad Maker plugin.

## [1.2.2] - 2026-11-02

### Changed

- The Atria Ad Maker logo is larger and its "Ad Maker" wordmark is bold, so
  the name stays readable at listing-icon size.
- Atria Ad Maker moves to 0.1.2. The Atria plugin moves to 1.2.2 with this
  release; its skills and MCP connection are unchanged.

## [1.2.1] - 2026-11-02

### Changed

- Atria Ad Maker has its own logo: the Atria rabbit with an "Ad Maker"
  wordmark, so it no longer looks identical to the Atria plugin. The Codex
  composer icon stays the rabbit alone, because the wordmark is unreadable at
  that size.
- Atria Ad Maker moves to 0.1.1. The Atria plugin moves to 1.2.1 with this
  release; its skills and MCP connection are unchanged.

## [1.2.0] - 2026-11-02

### Added

- Atria Ad Maker (`atria-ad-maker` 0.1.0), a separate plugin in the same
  marketplace. It connects to `https://api.tryatria.com/mcp/ad-maker` to create
  image and video ads from a brief, script or media, and to edit video projects
  with Raya. Generation uses workspace credits. It bundles no skills.
- README instructions for installing Atria Ad Maker in Claude Code and Codex.

### Changed

- The Atria plugin moves to 1.2.0 with this release; its skills and MCP
  connection are unchanged.

## [1.1.0] - 2026-10-13

### Changed

- Upgrade Inspo brand discovery, competitor research, and swipe-file workflows.
- Refine ad-account reporting and permission handling.
- Improve connection checks and Skill Hub compatibility.
- Update MCP Registry metadata to version 1.1.0.

## [1.0.0] - 2026-09-24

### Added

- Remote Atria MCP server declaration (`https://api.tryatria.com/mcp`, OAuth).
- Bundled skills: `setup`, `skill-hub`, `account-weekly-report`,
  `competitor-teardown`, `swipe-file`.
- Skills published to the Skill Hub and loaded through `skill-hub`:
  `creative-breakdown`, `competitive-landscape`, `creative-brief`,
  `ad-writer`, `brand-onboarding`. They port the methodology of Atria's own
  agent skills onto the MCP tool set.
- Marketplace manifests for Claude Code (`.claude-plugin/marketplace.json`) and
  Codex / ChatGPT desktop (`.agents/plugins/marketplace.json`).
- `server.json`, the record published to the official MCP Registry as
  `com.tryatria/atria`.
- A README that explains how to install the plugin in Claude, Claude Code,
  the ChatGPT desktop app and Codex CLI, or how to connect only the MCP server.
- `SECURITY.md` with the vulnerability reporting address.
