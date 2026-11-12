# BeReach MCP server

Describe your ideal customer in one sentence, or paste a LinkedIn post, and BeReach finds the people who match and
helps you write the first message. This repository connects BeReach to Claude, Claude Code, ChatGPT and any client
that speaks remote MCP.

- Server URL: `https://mcp.bereach.ai/mcp` (Streamable HTTP)
- Sign-in: OAuth in your browser. No API key to copy, a new account works too.
- Website: [bereach.ai](https://bereach.ai)

## What you can ask

- Find the people who match your ideal customer, by role, company, place or industry
- Find the companies that fit, and the people who work there
- Read a profile, or the people who reacted to and commented on a post
- Spot buying signals, such as a company hiring for a role
- Find work emails
- Prepare first messages in your own voice, and keep your list in one place

Ask in plain words, for example: who are the heads of sales at fintech companies in Germany with 50 to 200 people?

## Set up

### Claude (claude.ai and the desktop app)

Settings, Connectors, Add custom connector, then paste `https://mcp.bereach.ai/mcp` and sign in.
Step by step with pictures: [bereach.ai/linkedin-claude](https://bereach.ai/linkedin-claude).

### Claude Code

Install this repository as a plugin:

```
/plugin marketplace add bereach-ai/bereach-claude-plugin
/plugin install bereach@bereach
```

Or add the server directly:

```bash
claude mcp add --transport http bereach https://mcp.bereach.ai/mcp
```

Then type `/mcp`, pick bereach and sign in. More help: [bereach.ai/docs/claude-code](https://bereach.ai/docs/claude-code).

### Any other MCP client

```json
{
  "mcpServers": {
    "bereach": {
      "type": "http",
      "url": "https://mcp.bereach.ai/mcp"
    }
  }
}
```

## Skills

Six Claude skills for prospecting (personas, your voice, invitations, first messages, follow-ups, results at seven
days) live in their own repository:
[bereach-ai/linkedin-prospecting-skills](https://github.com/bereach-ai/linkedin-prospecting-skills).

## Pricing

14-day free trial. Then Pro at 99 € a month, or pay as you go from 20 €. You only pay for what is found.

## License

MIT

## Privacy

[bereach.ai/legal/privacy](https://bereach.ai/legal/privacy)
