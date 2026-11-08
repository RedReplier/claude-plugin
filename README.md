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

## What it does

Once installed, Claude or Hermes can:

- Add, update, analyze and remove monitored websites
- Add, reword, disable, enable and delete keywords. Keywords that fit the plan go active, the rest wait as pending
- List and count AI-scored mentions, filtered by relevance, minimum score, keyword, source or date
- Approve or reject mentions, or send them back to the inbox
- Explain why a mention scored the way it did, and draft a reply
- Turn email alerts on or off and set how often they arrive
- Work in any workspace your sign-in reaches

## Example

```
You: Any high-relevance mentions of my product this week?
Agent: 3 mentions scoring 75+. Top one: r/webdev, "Looking for an example tool"
       (score 85), someone asking for exactly what you offer, plus hits on
       Hacker News and Bluesky. Want me to approve it?
```

## Links

- Product: [redreplier.com](https://redreplier.com)
- AI agents: [redreplier.com/features/agents](https://redreplier.com/features/agents)
- Skill, MCP server and Gemini extension: [RedReplier/agent](https://github.com/RedReplier/agent)
- Privacy policy: [redreplier.com/privacy](https://redreplier.com/privacy)

## License

MIT
