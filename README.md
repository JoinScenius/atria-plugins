<h1 align="center">Atria Plugin</h1>

<p align="center">
  <b>Research ads and brands, analyze ad performance, and create image and video ads in your AI assistant.</b>
</p>

<p align="center">
  <a href="server.json"><img src="https://img.shields.io/badge/MCP_Registry-com.tryatria-000000?logo=modelcontextprotocol&logoColor=white" alt="MCP Registry: com.tryatria"></a>
  <a href="https://modelcontextprotocol.io"><img src="https://img.shields.io/badge/Model_Context_Protocol-compatible-000000" alt="Model Context Protocol compatible"></a>
  <img src="https://img.shields.io/badge/Auth-OAuth_2.1-2EBC4F" alt="Auth: OAuth 2.1">
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-blue" alt="License: MIT"></a>
</p>

<p align="center">
  <a href="https://docs.tryatria.com"><b>Documentation</b></a> ·
  <a href="https://www.tryatria.com"><b>Atria</b></a>
</p>

---

This plugin connects your AI assistant to the hosted Atria MCP server at
`https://api.tryatria.com/mcp` and adds skills for the jobs people ask for most. This repository
holds the plugin and its skills; the server itself is not open source.

Work in one Atria workspace from research through creation: find reference ads, read your
ad-account results, generate and edit media, and build video projects with Raya, Atria's video
agent. Generation and some revisions use workspace credits.

## Install

You need an Atria account. The first time your assistant uses Atria, it opens a browser window for
you to sign in.

### Claude (web and desktop)

Plugins are available on Claude's paid plans (Pro, Max, Team and Enterprise).

1. Open **Customize** in the left sidebar and go to the **Plugins** tab.
2. Next to **Personal plugins**, click **+** and choose **Add marketplace**.
3. Choose **Add from a repository** and enter `https://github.com/JoinScenius/atria-plugins`.
4. Install **Atria** from that marketplace.

Plugins you install here also load in Claude Code when you sign in with the same account.

### Claude Code

```
/plugin marketplace add JoinScenius/atria-plugins
/plugin install atria@atria
```

### ChatGPT desktop

The plugin runs in the ChatGPT desktop app, in Work mode and in Codex. It is not available on
ChatGPT web or mobile.

1. Add the marketplace with the Codex CLI:
   ```
   codex plugin marketplace add JoinScenius/atria-plugins
   ```
2. Restart the ChatGPT desktop app.
3. Open **Plugins**, choose the **Atria** marketplace and install **Atria**.

On ChatGPT Business and Enterprise, a workspace admin can add it for everyone instead: open
**Workspace settings**, go to **Plugins**, choose **Add** then **Import marketplace**, and enter
`https://github.com/JoinScenius/atria-plugins` as the source.

### Codex CLI

```
codex plugin marketplace add JoinScenius/atria-plugins
codex plugin add atria@atria
```

### Just the MCP server

To connect the server without the skills, point any MCP client at `https://api.tryatria.com/mcp`
(Streamable HTTP, OAuth, no API key). On ChatGPT web, use the
[Atria app](https://chatgpt.com/plugins/plugin_asdk_app_6a28d089f0cc8191b65dff43933d0adb). The
server's MCP Registry record is [`server.json`](server.json).

## What's included

- **The Atria MCP server**: tools for the ad library, competitors, your ad accounts, ad boards and
  notes, plus image and video generation, media editing, and video projects. The full list is in
  the [documentation](https://docs.tryatria.com).
- **Skills**:
  - `setup` checks the connection and runs a read-only test. Run it first.
  - `account-weekly-report` reports on one of your ad accounts, week over week.
  - `competitor-teardown` shows what one advertiser is running and how long each ad has lasted.
  - `swipe-file` collects reference ads for a theme, format or market and files them to a board.
  - `skill-hub` loads more Atria skills when you need them, such as creative breakdowns, briefs and
    ad copy. Some of these need a paid Atria plan.

## Create and edit ads

**Start from a script with Raya.** Give your assistant a script and ask it to make a video ad.
The project page shows the script, elements, storyboard and cut as the work progresses. Open
the page to review progress and approve each checkpoint yourself. You can continue working on
the same project in Atria.

**Edit a project directly.** Create projects, define elements and shots, generate image and video
candidates, select takes, and assemble and render cuts. Reuse your uploaded assets and library
scripts or elements across projects.

Example requests:

- "Find reference ads for my brand and help me turn the strongest idea into a video script."
- "Estimate the cost of one product image from my uploaded photo. Ask me before generating it."
- "Open my video project, check its current progress, and show me what needs my approval."

Generation and some revisions consume workspace credits. Ask for an estimate before paid work;
the final charge can differ from the estimate. Project edits and take selection can change
existing work, so review the proposed changes before applying them.

## Atria Ad Maker

**Atria Ad Maker** is a focused connection to the same creative tools. Install it from the
same marketplace when you want image and video creation and project editing in a smaller
tool set. It connects independently to
`https://api.tryatria.com/mcp/ad-maker` and currently includes no bundled skills.

```
/plugin install atria-ad-maker@atria          # Claude Code
codex plugin add atria-ad-maker@atria         # Codex CLI
```

## Update

```
/plugin marketplace update atria            # Claude Code
codex plugin marketplace upgrade atria      # Codex CLI and ChatGPT desktop
```

## Support

- Questions and bugs: [engineer-support@tryatria.com](mailto:engineer-support@tryatria.com), or open
  an issue.
- Security reports: see [SECURITY.md](SECURITY.md).
- [Privacy policy](https://www.tryatria.com/privacy-policy) ·
  [Terms of service](https://www.tryatria.com/terms-of-service)

This repository is MIT licensed. The Atria service is governed by its
[terms of service](https://www.tryatria.com/terms-of-service).
