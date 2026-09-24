# Changelog

All notable changes to the Atria plugin are recorded here. The format follows
[Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow semver.

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
