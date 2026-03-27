# you.com Web Search for OpenClaw

A web search skill for OpenClaw with three tools: free search, paid research, and paid content extraction.

## Features

- **Free search** — no API key required
- **Research AI** — synthesized answers with citations
- **Content extraction** — pull clean content from any URL
- **Cost controls** — paid endpoints require explicit user approval
- **Python3 parsing** — no jq dependency required

## Tools

| Tool | Cost | When to use |
|------|------|-------------|
| `youcom_search` | Free | Default web searches |
| `youcom_research` | Paid | Deep research with citations |
| `youcom_extract` | Paid | Content extraction from URLs |

## Installation

### Manual

Copy the skill to your OpenClaw workspace:

```bash
mkdir -p ~/.openclaw/workspace/skills/youcom-web-search
# Copy youcom-web-search/SKILL.md to that directory
```

### ClawHub

```bash
openclaw skills add youcom-web-search
```

## Configuration

### Free Tier (search)

No configuration needed. `youcom_search` works immediately.

### Paid Tier (research, extract)

1. Get your API key at https://you.com/platform/api-keys
2. Add to `~/.openclaw/.env`:

```
YOUCOM_API_KEY=your_key_here
```

3. Restart the gateway:

```bash
systemctl --user restart openclaw-gateway
```

## Usage

The skill automatically uses the right tool based on the request:

```
User: "search for openclaw github"
  → youcom_search (free, no approval needed)

User: "do deep research on AI agents"
  → youcom_research (PAID — asks for approval first)

User: "extract content from https://example.com"
  → youcom_extract (PAID — asks for approval first)
```

## Cost Control

This skill is designed to prevent runaway API spend:

- **Search is always free** — no API key needed
- **Research and extract require explicit approval** — the agent asks before using paid endpoints
- **Clear escalation path** — use free search first, only escalate to paid when truly needed

## Testing

```bash
# Test search (no API key needed)
curl -s "https://api.you.com/v1/agents/search?query=openclaw&num_results=5"

# Test research (requires API key)
curl -s -X POST "https://api.you.com/v1/agents/research" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YOUCOM_API_KEY" \
  -d '{"query": "openclaw setup", "depth": "standard"}'
```

## Requirements

- OpenClaw
- python3 (for JSON parsing — jq not required)

## License

MIT-0 — free to use, modify, and redistribute.
