# BeReach — Claude Code Plugin

LinkedIn outreach automation via [BeReach](https://berea.ch). Gives Claude access to 33 LinkedIn tools via MCP, plus a behavioral skill for campaign management.

## Install

```bash
claude plugin install bereach/claude-code-plugin
```

Or test locally:

```bash
claude --plugin-dir ./apps/web2/plugins/claude-code
```

## Setup

### 1. Set your API key

The plugin ships with `.mcp.json` which auto-configures the MCP server. You just need to set the `BEREACH_API_KEY` environment variable:

```bash
export BEREACH_API_KEY=brc_your_token_here
```

Or add it to your shell profile (`~/.zshrc`, `~/.bashrc`).

### 2. MCP config (auto-configured)

The plugin includes `.mcp.json` at its root — Claude Code detects it automatically. No manual MCP setup needed.

If you need manual config (e.g. for `claude_desktop_config.json`):

```json
{
  "mcpServers": {
    "bereach": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote@0.1.25",
        "https://mcp.bereach.dev/sse",
        "--header",
        "api-token:${BEREACH_API_KEY}"
      ],
      "env": {
        "BEREACH_API_KEY": "brc_your_token_here"
      }
    }
  }
}
```

## Usage

Type `/bereach:bereach` to see quick actions, or just ask Claude to:

- Check your credits or rate limits
- Search LinkedIn profiles or posts
- Visit a profile and get contact data
- Start a lead magnet campaign

The BeReach skill is loaded automatically and teaches Claude campaign workflows, pacing rules, and anti-hallucination constraints.

## What's included

| Component | Description |
| --- | --- |
| `.mcp.json` | MCP server auto-configuration |
| `commands/bereach.md` | `/bereach` slash command |
| `skills/bereach/SKILL.md` | Main behavioral skill |
| `skills/bereach/sub/lead-magnet.md` | Lead magnet workflow |
| `skills/bereach/sdk-reference.md` | SDK method reference |
