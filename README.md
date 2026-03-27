# you.com for OpenClaw

A privacy-respecting, Cloudflare-bypassing web search skill for OpenClaw with built-in cost controls.

## Features

- **Free search** — no API key required
- **Bypasses Cloudflare** — gets results from sites SearXNG struggles with (Medium, GitHub, etc.)
- **Cost controls** — paid endpoints (research, extract) require explicit user approval
- **Python3 parsing** — no jq dependency

## Tools

| Tool | Cost | When to use |
|------|------|-------------|
| `ydotcom_search` | Free | Default web searches |
| `ydotcom_research` | $0.005 + tokens | Deep research (user approval required) |
| `ydotcom_extract` | $1/1000 pages | Content extraction (user approval required) |

## Installation

### Option 1: ClawHub (recommended)

```bash
openclaw skills add edwardirby/youdotcom
```

### Option 2: Manual

Copy `youdotcom/SKILL.md` to your workspace skills directory:

```bash
mkdir -p ~/.openclaw/workspace/skills/youdotcom
cp SKILL.md ~/.openclaw/workspace/skills/youdotcom/SKILL.md
```

## Configuration

### Free Tier (search)

No configuration needed. `ydotcom_search` works immediately.

### Paid Tier (research, extract)

1. Get your API key at https://you.com/platform/api-keys
2. Add to `~/.openclaw/.env`:

```
YDC_API_KEY=your_key_here
```

3. Restart the gateway:

```bash
systemctl --user restart openclaw-gateway
```

## Usage

The skill automatically uses the right tool based on the request:

```
User: "search for openclaw github"
  → ydotcom_search (free, no approval needed)

User: "do deep research on AI agents"
  → ydotcom_research (PAID — asks for approval first)

User: "extract content from https://example.com"
  → ydotcom_extract (PAID — asks for approval first)
```

## Cost Control

This skill is designed to prevent runaway API spend:

- **Search is always free** — no API key needed, no credit consumed
- **Research and extract require explicit approval** — the agent is instructed to ask before using paid endpoints
- **Clear escalation path** — use free search first, only escalate to paid when truly needed

You get $100 in free credit for paid endpoints. Use it wisely.

## Escalation Chain

```
SearXNG (primary, local, free)
    ↓ (blocked/captcha)
you.com ydotcom_search (free, Cloudflare-bypassing)
    ↓ (if user explicitly asks for deep research)
you.com ydotcom_research (PAID)
    ↓ (if user explicitly asks to extract content)
you.com ydotcom_extract (PAID)
```

## Testing

```bash
# Test search (no API key needed)
curl -s "https://api.you.com/v1/agents/search?query=openclaw&num_results=5"

# Test research (requires API key)
curl -s -X POST "https://api.you.com/v1/agents/research" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YDC_API_KEY" \
  -d '{"query": "openclaw setup", "depth": "standard"}'
```

## Compared to Other Search Tools

| Feature | SearXNG | you.com | Tavily |
|---------|----------|----------|--------|
| Cost | Free (self-hosted) | Free / Paid | Paid API |
| Cloudflare bypass | ❌ | ✅ | ✅ |
| Local | ✅ | ❌ | ❌ |
| Research AI | ❌ | ✅ | ✅ |
| Content extraction | ❌ | ✅ | ✅ |
| Setup effort | Medium (Docker) | Low (API key) | Medium (API key) |

## Requirements

- OpenClaw
- python3 (for JSON parsing — jq not required)

## License

MIT-0 — free to use, modify, and redistribute.
