---
name: youdotcom
description: you.com web search, research, and content extraction tools with cost controls.
metadata:
  { "openclaw": { "emoji": "🔎", "requires": {} } }
---

# you.com Tools

## Overview

you.com provides three tools with a clear cost hierarchy:

| Tool | Cost | When to use |
|------|------|-------------|
| `ydotcom_search` | **Free** | Default for all web searches |
| `ydotcom_research` | **Paid** ($0.005 + tokens) | Only when explicitly requested |
| `ydotcom_extract` | **Paid** ($1/1000 pages) | Only when explicitly requested |

**IMPORTANT:** Research and extract tools consume your `$100 free credit`. Only use them when the user explicitly asks for "deep research", "comprehensive analysis", or "extract content from URLs".

## Cost Control Rules

### Free Tier (ydotcom_search)
- No API key needed
- No credit consumed
- Use for: all general web searches

### Paid Tier — ALWAYS confirm first
**Before using `ydotcom_research` or `ydotcom_extract`, you MUST:**
1. Tell the user which endpoint will be called and the approximate cost
2. Wait for explicit user approval
3. Only proceed if user confirms "yes" or equivalent

### Escalation Order
```
SearXNG (free, local)
    ↓ (if blocked/captcha/cloudflare)
you.com ydotcom_search (free)
    ↓ (if user explicitly asks for deep research)
you.com ydotcom_research (PAID — requires approval)
    ↓ (if user explicitly asks to extract page content)
you.com ydotcom_extract (PAID — requires approval)
```

## ydotcom_search

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

## ydotcom_research

**PAID — Confirm with user first.**

Synthesized, citation-backed deep research on a topic. Uses your `$100 free credit`.

```bash
QUERY="$1"
DEPTH="${2:-standard}"
curl -s -X POST "https://api.you.com/v1/agents/research" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YDC_API_KEY" \
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

**IMPORTANT:** Check `$YDC_API_KEY` is set before calling. If not set, tell user research requires an API key.

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

## ydotcom_extract

**PAID — Confirm with user first.**

Extract clean content from specific URLs. Uses your `$100 free credit`.

```bash
URLS="$1"
curl -s -X POST "https://ydc-index.io/v1/contents" \
  -H "Content-Type: application/json" \
  -H "X-API-Key: $YDC_API_KEY" \
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

**IMPORTANT:** Check `$YDC_API_KEY` is set before calling. If not set, tell user content extraction requires an API key.

### When to Use Extract (paid)
✅ Use when user says:
- "extract content from..."
- "read this page..."
- "get the full article from..."
- "scrape this URL..."

❌ Do NOT use when:
- A simple search returns the answer
- User just wants to know what's on a page
- ydotcom_search results already answered the question

## Getting Your API Key

1. Go to https://you.com/platform/api-keys
2. Create a new API key
3. Add to `~/.openclaw/.env`: `YDC_API_KEY=your_key_here`
4. Restart the gateway for changes to take effect

## Quick Reference

| Need | Tool | Cost | API Key |
|------|------|------|---------|
| Any web search | `ydotcom_search` | Free | ❌ |
| Deep research | `ydotcom_research` | Paid | ✅ |
| Extract page content | `ydotcom_extract` | Paid | ✅ |
