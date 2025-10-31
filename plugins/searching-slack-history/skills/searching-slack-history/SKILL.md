---
name: searching-slack-history
description: Use when needing to search Slack message history, find past conversations, or retrieve channel messages - provides Slack Web API patterns for search.messages and conversations.history endpoints using WebFetch or curl
---

# Searching Slack History

## Overview

Search Slack message history using the Slack Web API.

**Core principle:** Use Slack's REST API endpoints with WebFetch or curl to search messages, retrieve channel history, and find past conversations.

**Prerequisites:** Slack Bot token with appropriate scopes (setup guide below)

## Setup Guide (One-Time)

### Creating a Slack Bot Token

1. Visit https://api.slack.com/apps → Click "Create New App"
2. Choose "From scratch"
3. Name it (e.g., "Claude Search Bot")
4. Select your workspace
5. Navigate to "OAuth & Permissions" in sidebar
6. Scroll to "Scopes" → "Bot Token Scopes"
7. Add these scopes:
   - `search:read` - Search messages across workspace
   - `channels:history` - Read public channel history
   - `channels:read` - List public channels
   - `groups:history` - Read private channel history (if needed)
   - `groups:read` - List private channels (if needed)
8. Click "Install to Workspace" at top of page
9. Authorize the app
10. Copy the "Bot User OAuth Token" (starts with `xoxb-`)
11. Store securely as environment variable:
    ```bash
    export SLACK_TOKEN="xoxb-your-token-here"
    ```

**Security:** Never commit tokens to git. Use environment variables or secure credential storage.

**Private channels:** Bot must be invited to private channels before it can search them (`/invite @YourBotName`).

## Quick Reference

| Task | Endpoint | Key Parameters |
|------|----------|----------------|
| Search all messages | `search.messages` | `query`, `sort`, `count` |
| Get channel history | `conversations.history` | `channel`, `limit`, `oldest` |
| Find channel ID | `conversations.list` | `types`, `exclude_archived` |
| Search by date range | `search.messages` | `query` with date operators |

## API Patterns

### Search Messages Globally

**Endpoint:** `POST https://slack.com/api/search.messages`

**Use case:** Find messages across all channels the bot has access to.

**Example using Bash with curl:**
```bash
curl -X POST https://slack.com/api/search.messages \
  -H "Authorization: Bearer $SLACK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "your search terms",
    "count": 20,
    "sort": "timestamp"
  }'
```

**Example using WebFetch (Claude Code):**
```typescript
// Note: WebFetch is for fetching web content, not making API calls
// For API calls with authentication, use Bash with curl instead
```

**Query operators:**
- `"exact phrase"` - Match exact phrase
- `from:@username` - Messages from specific user
- `in:#channel` - Messages in specific channel
- `after:YYYY-MM-DD` - Messages after date
- `before:YYYY-MM-DD` - Messages before date

**Example with operators:**
```json
{
  "query": "deployment after:2025-01-01 in:#engineering",
  "count": 50
}
```

**Rate limit:** Tier 3 (20 requests/minute)

### Get Channel History

**Endpoint:** `POST https://slack.com/api/conversations.history`

**Use case:** Retrieve recent messages from a specific channel.

**Important:** Requires channel ID, not channel name. Use `conversations.list` first if you only have the name.

**Example:**
```bash
curl -X POST https://slack.com/api/conversations.history \
  -H "Authorization: Bearer $SLACK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "channel": "C1234567890",
    "limit": 100
  }'
```

**Time filtering:**
```json
{
  "channel": "C1234567890",
  "oldest": "1609459200.000000",
  "latest": "1640995200.000000",
  "limit": 100
}
```

Timestamps are Unix epoch time with microseconds (e.g., `1609459200.000000` = 2021-01-01).

### Find Channel ID from Name

**Endpoint:** `POST https://slack.com/api/conversations.list`

**Use case:** Get channel ID when you only know the channel name.

**Example:**
```bash
curl -X POST https://slack.com/api/conversations.list \
  -H "Authorization: Bearer $SLACK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "types": "public_channel,private_channel",
    "exclude_archived": true
  }'
```

**Filter response:** Look for channel with matching `name` field, extract `id`.

### Handling Pagination

All endpoints return paginated results. Check for `response_metadata.next_cursor`:

```bash
# First request
curl ... -d '{"query": "search", "cursor": ""}'

# If response contains "next_cursor": "dXNlcjpVMDYxTkZUVDI="
# Second request
curl ... -d '{"query": "search", "cursor": "dXNlcjpVMDYxTkZUVDI="}'
```

Continue until `next_cursor` is empty or not present.

## Common Mistakes

### 1. Missing Token Scopes
**Symptom:** `{"ok": false, "error": "missing_scope"}`

**Fix:** Add required scopes in Slack App OAuth settings, then reinstall app to workspace.

### 2. Using Channel Names Instead of IDs
**Symptom:** `{"ok": false, "error": "channel_not_found"}`

**Fix:** Call `conversations.list` first to map channel name → ID.

### 3. Ignoring Pagination
**Symptom:** Only getting first 20-100 results, missing older messages.

**Fix:** Check for `response_metadata.next_cursor` and make subsequent requests with cursor parameter.

### 4. Rate Limit Errors
**Symptom:** `{"ok": false, "error": "rate_limited"}` or HTTP 429

**Fix:** Implement exponential backoff. Wait time often provided in `Retry-After` header.

### 5. Private Channel Access
**Symptom:** Empty results or permission errors for private channels.

**Fix:**
- Add `groups:history` and `groups:read` scopes
- Invite bot to private channel: `/invite @YourBotName`

### 6. Token Exposure
**Risk:** Hardcoded tokens in scripts or commits give workspace access to anyone.

**Fix:**
- Always use environment variables: `$SLACK_TOKEN`
- Add `.env` to `.gitignore`
- Rotate token immediately if exposed

## Response Format

Successful responses have `"ok": true`:

```json
{
  "ok": true,
  "messages": {
    "matches": [
      {
        "type": "message",
        "user": "U1234567890",
        "text": "Message content",
        "ts": "1234567890.123456",
        "permalink": "https://workspace.slack.com/archives/...",
        "channel": {
          "id": "C1234567890",
          "name": "general"
        }
      }
    ],
    "total": 150
  },
  "response_metadata": {
    "next_cursor": "dXNlcjpVMDYxTkZUVDI="
  }
}
```

Error responses have `"ok": false`:
```json
{
  "ok": false,
  "error": "invalid_auth"
}
```

## Real-World Workflow

**Scenario:** Find all messages about "deployment" in #engineering from last week.

```bash
# 1. Get channel ID
curl -X POST https://slack.com/api/conversations.list \
  -H "Authorization: Bearer $SLACK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"types": "public_channel"}' \
  | jq -r '.channels[] | select(.name=="engineering") | .id'
# Returns: C1234567890

# 2. Calculate timestamps (Unix epoch)
# 7 days ago: date -d "7 days ago" +%s
# Today: date +%s

# 3. Search messages
curl -X POST https://slack.com/api/search.messages \
  -H "Authorization: Bearer $SLACK_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "query": "deployment in:#engineering after:2025-10-24",
    "count": 100,
    "sort": "timestamp"
  }'
```

## Further Reading

- Slack Web API Documentation: https://api.slack.com/web
- search.messages: https://api.slack.com/methods/search.messages
- conversations.history: https://api.slack.com/methods/conversations.history
- OAuth Scopes Reference: https://api.slack.com/scopes
