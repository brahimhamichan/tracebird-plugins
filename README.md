<p align="center"><img src="plugins/tracebird/assets/logo.png" width="96" alt="Tracebird"></p>

# Tracebird for AI agents

Run your [Tracebird](https://tracebird.io) workspace from Claude Code, OpenAI Codex or Grok Build: Facebook and Google Ads performance with attributed and front-end/back-end revenue, automation rules, split tests, tracking, reports and settings. The plugin adds the Tracebird MCP server (sign in with OAuth, no API key) and skills that know how to use it well.

## Install

One command for every agent on your machine:

```sh
curl -fsSL https://tracebird.io/install.sh | sh
```

Or per agent:

| Agent | Install | Sign in |
|---|---|---|
| Claude Code | `claude plugin install tracebird --marketplace brahimhamichan/tracebird-plugins` | In Claude Code: `/mcp` → `plugin:tracebird:tracebird` → Authenticate |
| OpenAI Codex | `codex plugin marketplace add brahimhamichan/tracebird-plugins && codex plugin add tracebird@tracebird` | `codex mcp login tracebird` |
| Grok Build | `grok plugin install brahimhamichan/tracebird-plugins#plugins/tracebird --trust` | In Grok: `/mcps` → tracebird → press `i` |

Inside a Claude Code session you can also run `/plugin install tracebird --marketplace brahimhamichan/tracebird-plugins`.

Signing in opens Tracebird in your browser: pick the workspace, choose whether the agent may make changes, and click Allow. You can disconnect it anytime in Tracebird under Settings → API & MCP.

## What you get

- **MCP server** `https://tracebird.io/mcp`: every action the Tracebird app offers (222 tools), with the same permissions as your account.
- **Skills**:

| Skill | Use it to |
|---|---|
| `getting-started` | Connect, check integrations and learn what Tracebird can do |
| `performance-review` | Review spend, ROAS, CPA and front-end/back-end revenue for any period |
| `cut-wasted-spend` | Find ads and ad sets burning budget and pause or rebalance them (with your confirmation) |
| `build-automation` | Design, preview and turn on automation rules safely |
| `split-testing` | Run landing-page split tests on your own link domains and call winners |
| `tracking-setup` | Set up the pixel, CRMs, webhooks, conversions, Meta CAPI and revenue settings |
| `reports-and-dashboards` | Build reports, schedules, dashboards and embeds; sync data to your apps |
| `workspace-admin` | Manage workspace settings, team, notifications, logs and webhooks |

Agents always ask before changing live ads, budgets, automations or settings.

## Other clients

Any MCP client can connect to `https://tracebird.io/mcp` (Streamable HTTP, MCP 2026-07-28 with fallback to earlier versions, OAuth 2.1). Claude and ChatGPT: add it as a custom connector. Cursor and VS Code: one-click buttons in Tracebird under Settings → API & MCP. REST API and docs: [tracebird.io/docs/api](https://tracebird.io/docs/api).

## Repository layout

```
.claude-plugin/marketplace.json     Claude Code and Grok Build marketplace
.agents/plugins/marketplace.json    Codex marketplace
plugins/tracebird/
  .claude-plugin/plugin.json        Claude Code / Grok Build manifest
  .codex-plugin/plugin.json         Codex manifest
  plugin.json, mcp.json             Agent Plugins 1.0 (portable) manifest and MCP config
  .mcp.json                         MCP server for Claude Code, Codex and Grok Build
  skills/<name>/SKILL.md            Agent Skills, shared by every agent
```

Support: support@tracebird.io · [Privacy](https://tracebird.io/privacy) · [Terms](https://tracebird.io/terms)
