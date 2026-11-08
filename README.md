# RedReplier plugin for Claude Code and Hermes Agent

Find leads in public conversations on Reddit, Hacker News, X, Bluesky and Facebook from Claude Code or Hermes Agent. RedReplier watches for your keywords and scores every mention 0 to 100 for relevance, so the agent can surface the real leads and skip the noise.

## What this repo is

This repo is the RedReplier plugin for Claude Code and Hermes Agent, and nothing else. Both read the same package: `.claude-plugin/plugin.json`, `.mcp.json` and `.claude-plugin/marketplace.json` for Claude Code, `plugin.json` and `mcp.json` for Hermes, and one skill in [`skills/redreplier/`](./skills/redreplier). The plugin connects to the hosted MCP server at `https://mcp.redreplier.com/mcp`, signs in with OAuth and never asks for an API key. Its skill only calls MCP tools and ships no script, because the Claude plugin directory and the Hermes catalog both review for that.

Everything else lives in [RedReplier/agent](https://github.com/RedReplier/agent): the agent skill for `npx skills add` (a CLI script with an API key), the source of the MCP server, and the Gemini CLI extension.

| You use | Install | Sign-in |
| --- | --- | --- |
| Claude Code | `/plugin install redreplier --marketplace RedReplier/claude-plugin` | OAuth |
| Hermes Agent | `hermes plugins install redreplier` | OAuth |
| Gemini CLI | `gemini extensions install https://github.com/RedReplier/agent` | OAuth |
| Claude Desktop, Cursor, ChatGPT or any MCP client | Add `https://mcp.redreplier.com/mcp` as a remote server | OAuth, or an API key header |
| Cursor, Codex, Windsurf, Claude Code or any agent with skills | `npx skills add RedReplier/agent` | API key |

## Install in Claude Code

```
/plugin install redreplier --marketplace RedReplier/claude-plugin
```

On Claude Code older than 2.1.275, add the marketplace first:

```
/plugin marketplace add RedReplier/claude-plugin
/plugin install redreplier
```

## Setup

1. Create an account at [redreplier.com/signup](https://redreplier.com/signup) and add the websites you want to monitor.
2. In Claude Code, run `/mcp`, pick `redreplier` and sign in with the email you use on RedReplier.

The plugin talks to RedReplier only through its MCP server at `mcp.redreplier.com`, signed in with OAuth. It never asks for an API key and reads nothing from your environment or config files.

## Install in Hermes Agent

You need Hermes Agent 0.21.6 or newer and Node.js 20.18.1 or newer with `npx` on your `PATH`.

From the Hermes plugin catalog:

```bash
hermes plugins install redreplier
hermes plugins enable redreplier
```

Or straight from GitHub:

```bash
hermes plugins install RedReplier/claude-plugin --no-enable
hermes plugins enable redreplier
```

Start a new session. The first time the server starts, your browser opens the RedReplier sign-in page. Sign in with the email you use on RedReplier and approve. The token is kept and refreshed, so you sign in once. If the tools don't show up after you sign in, run `/reload-mcp`.

The Hermes package is `plugin.json`, `mcp.json` and the skill in `skills/redreplier/`. It ships no code of its own. Before you install, know that:

- `mcp.json` runs the pinned `mcp-remote@0.1.49` through `npx`, which downloads it from npm on first use. It opens a local callback port for the sign-in and stores the OAuth tokens in `~/.mcp-auth`.
- Every tool call goes to `mcp.redreplier.com` and acts on your RedReplier account, with your own role in each workspace. The server decides the tool set at runtime, so the pinned commit doesn't cover it.
- `plugin.json` marks the server `untrusted`. Hermes asks before every tool the server doesn't mark read-only (adding, changing or deleting websites and keywords, approving or rejecting mentions, explaining a mention's score and changing alert settings), and refuses those tools in cron and other unattended runs.
- Keywords count against your RedReplier plan, and `analyze_website` reads the website's public pages.
- The package sends no telemetry.

On a server or another machine without a browser, skip the plugin and add the server to `~/.hermes/config.yaml` with an API key from [redreplier.com/api-tokens](https://redreplier.com/api-tokens):

```yaml
mcp_servers:
  redreplier:
    url: "https://mcp.redreplier.com/mcp"
    headers:
      Authorization: "Bearer ${REDREPLIER_API_KEY}"
```

Put `REDREPLIER_API_KEY=...` in `~/.hermes/.env`. The key works like a password for your RedReplier account, so don't paste it into a chat.

## Example prompts

- "Show me this week's best leads, score 70 and above."
- "Why did this Reddit thread score 85? Draft a reply I can post."
- "Add 'social media scheduler' and 'buffer alternative' as keywords for my website."
- "Approve the top three mentions and reject the rest."
- "How many Hacker News mentions did we get this month?"
- "Send me an email digest every 4 hours."
- "Write a description for https://example.com and start monitoring it."

## Tools

19 tools. Mentions come from Reddit, Hacker News, X, Bluesky and Facebook. Slack, Telegram, Notion and Linear alerts are set up in the RedReplier app.

### Workspaces and websites

| Tool | What it does | Changes data |
| --- | --- | --- |
| `list_workspaces` | List the workspaces the sign-in can act in; pass an id as `workspaceId` to any other tool | No |
| `list_websites` | List monitored websites with their keywords and statuses | No |
| `get_website` | Get one website with its keywords | No |
| `create_website` | Add a website to monitor, with optional keywords and description | Yes |
| `update_website` | Change a website's name or description | Yes |
| `delete_website` | Stop monitoring a website | Yes |
| `analyze_website` | Write a description of a website from its URL with AI | Yes |

### Keywords

| Tool | What it does | Changes data |
| --- | --- | --- |
| `add_keywords` | Add keywords to a website; those that fit the plan go active, the rest wait as pending | Yes |
| `edit_keyword` | Reword a keyword; it is graded again and keeps its slot | Yes |
| `disable_keyword` | Stop monitoring a keyword; its mentions stay | Yes |
| `enable_keyword` | Turn a disabled keyword back on; it waits as pending when the plan is full, and never charges | Yes |
| `delete_keyword` | Delete a keyword and every mention it produced | Yes |
| `keyword_change_usage` | Keyword-edit allowance and usage | No |

### Mentions

| Tool | What it does | Changes data |
| --- | --- | --- |
| `list_mentions` | List AI-scored mentions, filtered by status, relevance, minimum score, keyword, source and date | No |
| `count_mentions` | Count mentions with the same filters | No |
| `update_mention_status` | Approve, reject or send a mention back to the inbox | Yes |
| `explain_mention` | Why a mention scored the way it did, with tags and a drafted reply | Yes |

### Alerts

| Tool | What it does | Changes data |
| --- | --- | --- |
| `get_alert_settings` | Email alert settings: on or off, cadence and the plan's fastest cadence | No |
| `update_alert_settings` | Turn email alerts on or off and set the cadence | Yes |

## Roles, permissions and safety

- Every call runs with the signed-in member's role in the workspace. Reads need `workspace.read`; every change needs `redreplier.write`, which Admin and Editor hold.
- Read-only tools carry `readOnlyHint`, so clients that confirm writes ask before adding websites or keywords, triaging mentions or changing alerts.
- Keywords count against your RedReplier plan. Adding one never charges: keywords beyond the plan wait as pending.
- Mentions are public posts. `analyze_website` reads the website's public pages.

## FAQ

**Do I need an API key?**
No. Point your client at `https://mcp.redreplier.com/mcp` and sign in with the email you use on RedReplier. Keys are for headless agents and clients without OAuth.

**Can the agent post replies for me?**
No. It drafts a reply with `explain_mention`; you post it yourself.

**Will adding keywords charge me?**
No. Keywords that fit the plan go active, the rest wait as pending until you upgrade in the app.

**Which sources does it watch?**
Reddit, Hacker News, X (Twitter), Bluesky and Facebook.

**Can I use RedReplier outside Claude Code and Hermes?**
Yes. Claude Desktop, ChatGPT, Cursor, Windsurf, VS Code, Codex and Gemini CLI connect to the same server at `https://mcp.redreplier.com/mcp`. The setup for each is in [RedReplier/agent](https://github.com/RedReplier/agent#setup-by-client).

## Links

- Product: [redreplier.com](https://redreplier.com)
- AI agents: [redreplier.com/features/agents](https://redreplier.com/features/agents)
- Skill, MCP server and Gemini extension: [RedReplier/agent](https://github.com/RedReplier/agent)
- Privacy policy: [redreplier.com/privacy](https://redreplier.com/privacy)

## License

MIT
