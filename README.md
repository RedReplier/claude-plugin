# RedReplier plugin for Claude Code and Hermes Agent

Monitor **Reddit, Hacker News, X, Bluesky, and Facebook** for mentions of your product from Claude Code. RedReplier AI-scores every mention 0-100 for relevance, so Claude can surface the real leads and skip the noise.

## Install in Claude Code

```
/plugin marketplace add redreplier/claude-plugin
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

Once installed, Claude can:

- **Manage monitored websites** — add, update, analyze (AI description), remove
- **Manage keywords** — add, edit, enable/disable, and delete; keywords beyond the plan stay pending
- **Triage mentions** — list AI-scored mentions from Reddit, Hacker News, X, Bluesky, and Facebook, filter by relevance/minimum score/keyword/source/date, approve or reject leads
- **Explain relevance** — see *why* a mention was scored the way it was
- **Configure alerts** — enable email digests and set the cadence
- **Switch workspaces** — one sign-in reaches every workspace you belong to

## Example

```
You: Any high-relevance mentions of my product this week?
Claude: 3 mentions scoring 75+. Top one: r/webdev — "Looking for an example tool"
        (score 85) — someone asking for exactly what you offer, plus hits on
        Hacker News and Bluesky. Want me to approve it?
```

## Alternative: MCP

For Claude Desktop, Cursor, or other MCP-compatible clients:

```json
{
  "mcpServers": {
    "redreplier": {
      "type": "http",
      "url": "https://mcp.redreplier.com/mcp",
      "headers": { "Authorization": "Bearer redreplier_your_key" }
    }
  }
}
```

## Links

- Product: [redreplier.com](https://redreplier.com)
- API Tokens: [redreplier.com/api-tokens](https://redreplier.com/api-tokens)
- MCP Server: [github.com/redreplier/mcp-server](https://github.com/redreplier/mcp-server)
- Privacy policy: [redreplier.com/privacy](https://redreplier.com/privacy)

## License

MIT
