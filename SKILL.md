---
name: brave-search
description: Search the web with the Brave Search API. Use when user asks for current information, external facts, documentation lookup, news, research, URLs, or web results beyond local files.
---

# Brave Search

Search web via Brave Search API. Use for current facts, docs, news, and external URLs. Prefer local repo search for project files.

## Auth

Read API key from:

1. `$BRAVE_SEARCH_API_KEY`
2. `~/.config/pi/brave-search-token.json` (`apiKey` or `accessToken` field)
3. legacy fallback `~/.config/pi/brave-search-token` (raw key)

Never print API key. Do not include it in final answers, code snippets, logs, or client bundles.

Token metadata file shape:

```json
{
  "apiKey": "...",
  "createdAt": "2026-04-27",
  "scopes": ["web_search"],
  "note": "Brave Search API key for Pi brave-search skill"
}
```

For 401/403, mention likely invalid/revoked key or Brave Search API plan/quota issue. Do not reveal key.

## Preferred helper

From this skill directory, run:

```bash
./scripts/brave-search.py "query" --count 5
```

The script:

- queries `https://api.search.brave.com/res/v1/web/search`
- writes raw API response to `/tmp/brave-search.json`
- writes compact summary to `/tmp/brave-search-summary.json`
- prints compact summary to stdout

Options:

```bash
./scripts/brave-search.py "query" --count 10
./scripts/brave-search.py "query" --country US --search-lang en
./scripts/brave-search.py "query" --freshness pw      # past week
./scripts/brave-search.py "query" --offset 10        # next page
./scripts/brave-search.py "query" --json             # compact JSON only
```

## Workflow

1. Use precise query terms.
2. Run helper.
3. Read compact summary first. Avoid dumping raw JSON into chat.
4. If results weak, refine query once or twice.
5. When answering from web, cite result URLs or mention source names.
6. If user needs full page content, say Brave result snippets may be insufficient and fetch/inspect the specific URL only if tools/network allow.

## Manual fetch

```bash
TOKEN="${BRAVE_SEARCH_API_KEY:-$(python3 - <<'PY'
import json, os
p=os.path.expanduser('~/.config/pi/brave-search-token.json')
try:
  data=json.load(open(p))
  print(data.get('apiKey') or data.get('accessToken') or '')
except Exception:
  p=os.path.expanduser('~/.config/pi/brave-search-token')
  print(open(p).read().strip() if os.path.exists(p) else '')
PY
)}"

curl -sS \
  -H "Accept: application/json" \
  -H "X-Subscription-Token: $TOKEN" \
  --get "https://api.search.brave.com/res/v1/web/search" \
  --data-urlencode "q=SEARCH TERMS" \
  --data-urlencode "count=5" \
  > /tmp/brave-search.json
```

## Notes

- Brave Search API returns snippets, titles, URLs, metadata. It does not guarantee full page content.
- `count` max depends on API plan; keep 5-10 unless user needs more.
- Use `/tmp/brave-search-summary.json` for token-efficient inspection.
