# RedReplier — Claude Code Plugin

Monitor **Reddit, Hacker News, X, and Bluesky** for mentions of your product from Claude Code. RedReplier AI-scores every mention 0-100 for relevance, so Claude can surface the real leads and skip the noise.

## Install

```
/plugin marketplace add redreplier/claude-plugin
/plugin install redreplier
```

## Setup

1. Create an account at [redreplier.com/signup](https://redreplier.com/signup) and add the websites you want to monitor.
2. In Claude Code, run `/mcp`, pick `redreplier` and sign in with the email you use on RedReplier.

The plugin talks to RedReplier only through its MCP server at `mcp.redreplier.com`, signed in with OAuth. It never asks for an API key and reads nothing from your environment or config files.

## What it does

Once installed, Claude can:

- **Manage monitored websites** — add, update, analyze (AI description), remove
- **Manage keywords** — add, edit, enable/disable, and activate within your plan
- **Triage mentions** — list AI-scored mentions from Reddit, Hacker News, X, and Bluesky, filter by relevance/keyword/source/date, approve or reject leads
- **Explain relevance** — see *why* a mention was scored the way it was
- **Configure alerts** — enable email digests and set the cadence

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
