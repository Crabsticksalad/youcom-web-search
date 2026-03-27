---
name: youcom-web-search
description: you.com web search, research, and content extraction tools for OpenClaw.
metadata:
  { "openclaw": { "emoji": "🔎", "requires": {} } }
---

# you.com Tools for OpenClaw

## Overview

you.com provides three tools with a clear cost hierarchy:

| Tool | Cost | When to use |
|------|------|-------------|
| `youcom_search` | **Free** | Default for all web searches |
| `youcom_research` | **Paid** | Deep research with citations |
| `youcom_extract` | **Paid** | Content extraction from URLs |

**IMPORTANT:** Paid endpoints require API key. Only use them when the user explicitly asks for "deep research", "comprehensive analysis", or "extract content from URLs".

## Cost Control Rules

### Free Tier (youcom_search)
- No API key needed
- Use for: all general web searches

### Paid Tier — ALWAYS confirm first
**Before using `youcom_research` or `youcom_extract`, you MUST:**
1. Tell the user which endpoint will be called and the approximate cost
2. Wait for explicit user approval
3. Only proceed if user confirms

### Escalation Order
```
youcom_search (free)
    ↓ (if user explicitly asks for deep research)
youcom_research (PAID — requires approval)
    ↓ (if user explicitly asks to extract page content)
youcom_extract (PAID — requires approval)
```

## youcom_search

Use for all basic web searches. No API key, no cost.

```bash
QUERY="$1"
NUM="${2:-5}"
curl -s "https://api.you.com/v1/agents/search?query=$(echo "$QUERY" | sed 's/ /+/g')&num_results=$NUM" \
  -H "Accept: application/json" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for r in data.get('results', {}).get('web', []):
    print(r.get('title', 'N/A'))
    print(r.get('url', 'N/A'))
    print(r.get('description', 'N/A')[:200])
    print()
"
```

| Parameter | Description |
|-----------|-------------|
| `query` | Search query (positional, required) |
| `num_results` | Number of results (positional, default: 5, max: 10) |

**Example output:**
```
Result Title
https://example.com
Brief description of the page
```

### Usage Tips
- Supports search operators: `site:`, `filetype:`, `time:` (e.g., `site:reddit.com openclaw`)
- Returns up to 10 results per query
- No API key needed
- Uses python3 for JSON parsing

## youcom_research

**PAID — Confirm with user first.**

Synthesized, citation-backed deep research on a topic.

```bash
QUERY="$1"
DEPTH="${2:-standard}"
curl -s -X POST "https://api.you.com/v1/agents/research" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YOUCOM_API_KEY" \
  -d "{\"query\": \"$QUERY\", \"depth\": \"$DEPTH\"}" | python3 -c "
import json, sys
data = json.load(sys.stdin)
print(json.dumps(data, indent=2, ensure_ascii=False)[:5000])
"
```

| Parameter | Description |
|-----------|-------------|
| `query` | Research topic (positional, required) |
| `depth` | `lite`, `standard`, `deep`, `exhaustive` (positional, default: standard) |

**IMPORTANT:** Check `$YOUCOM_API_KEY` is set before calling. If not set, tell user research requires an API key.

### When to Use Research (paid)
✅ Use when user says:
- "do deep research on..."
- "comprehensive analysis of..."
- "thorough investigation of..."
- "write a detailed report on..."

❌ Do NOT use when:
- User just asks a question ("what is...?")
- Basic lookup ("when was X released?")
- User said "just search for..." or "look up..."

## youcom_extract

**PAID — Confirm with user first.**

Extract clean content from specific URLs.

```bash
URLS="$1"
curl -s -X POST "https://ydc-index.io/v1/contents" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YOUCOM_API_KEY" \
  -d "{\"urls\": [$URLS], \"highlights\": true}" | python3 -c "
import json, sys
data = json.load(sys.stdin)
for r in data:
    print('URL:', r.get('url', 'N/A'))
    print('Title:', str(r.get('title', 'N/A'))[:100])
    print('Content:', str(r.get('content', 'N/A'))[:500])
    print()
"
```

| Parameter | Description |
|-----------|-------------|
| `urls` | URLs as comma-separated string (positional, required) |

**IMPORTANT:** Check `$YOUCOM_API_KEY` is set before calling. If not set, tell user content extraction requires an API key.

### When to Use Extract (paid)
✅ Use when user says:
- "extract content from..."
- "read this page..."
- "get the full article from..."
- "scrape this URL..."

❌ Do NOT use when:
- A simple search returns the answer
- User just wants to know what's on a page
- youcom_search results already answered the question

## Getting Your API Key

1. Go to https://you.com/platform/api-keys
2. Create a new API key
3. Add to `~/.openclaw/.env`: `YOUCOM_API_KEY=your_key_here`
4. Restart the gateway: `systemctl --user restart openclaw-gateway`

## Quick Reference

| Need | Tool | Cost | API Key |
|------|------|------|---------|
| Any web search | `youcom_search` | Free | ❌ |
| Deep research | `youcom_research` | Paid | ✅ |
| Extract page content | `youcom_extract` | Paid | ✅ |
