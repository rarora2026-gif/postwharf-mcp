# PostWharf MCP server

Schedule and publish social media posts from Claude, ChatGPT, Cursor, Codex or any other MCP client.

PostWharf runs a hosted MCP server. Connect it to your agent and the agent can draft, schedule, publish and track posts for one brand, with a delivery receipt from each network a post goes to.

It publishes to **Instagram, TikTok, X, LinkedIn, YouTube, Facebook, Threads, Pinterest, Bluesky and Telegram**. Reddit, Discord, Snapchat and WordPress are request-only: PostWharf cannot publish to them yet.

There is nothing to install. The server is hosted, and this repository holds its documentation and its MCP Registry entry, not its source code.

| | |
|---|---|
| Endpoint | `https://mcp.postwharf.com/mcp` |
| Transport | Streamable HTTP |
| Auth | `Authorization: Bearer <workspace key>`, or a connect link for ChatGPT and the Claude apps |
| MCP Registry | `com.postwharf/mcp` ([server.json](server.json) is a copy of the entry) |
| Docs | https://postwharf.com/developers |

## Authentication

Every connection acts for one workspace, and a workspace is one brand. The agent sees that brand's accounts and no others, so if you run five brands you add five connections.

There are two kinds of credential:

- **Workspace key.** Send it as `Authorization: Bearer <workspace key>` from clients that let you set a header: Claude Code, Cursor and Codex. Find it in the dashboard under API keys.
- **Connect link.** ChatGPT and the Claude web and desktop apps have no field for a header, so they use a link instead. Copy it from Agent & MCP in the dashboard, paste it as the server URL, and set no authentication: the link is the key. It is separate from your workspace key, so if it ends up somewhere it should not, you can replace it from the dashboard and the CLI and anything scripted keep running.

## Setup

### Claude Code

```bash
claude mcp add --transport http \
  wharf https://mcp.postwharf.com/mcp \
  --header "Authorization: Bearer $POSTWHARF_KEY"
```

`$POSTWHARF_KEY` is your workspace key. Keep it in your environment rather than in a file a repository tracks.

### Cursor

```json
{
  "mcpServers": {
    "wharf": {
      "url": "https://mcp.postwharf.com/mcp",
      "headers": { "Authorization": "Bearer YOUR_WORKSPACE_KEY" }
    }
  }
}
```

### Codex

```text
# Point the connector at:
https://mcp.postwharf.com/mcp
# with the header:
Authorization: Bearer YOUR_WORKSPACE_KEY
```

### Claude on the web or desktop

```text
# Settings -> Connectors -> Add custom connector
# Server URL: the connect link on your PostWharf dashboard,
#             under Agent & MCP -> Claude app
# Leave the OAuth fields empty. The link is the key.
```

### ChatGPT

```text
# Settings -> Connectors -> Create
# Server URL: the connect link on your PostWharf dashboard,
#             under Agent & MCP -> ChatGPT
# Authentication: None. The link is the key.
# Developer mode has to be on, and it is not on the free plan.
```

### Any other client

Any client that speaks Streamable HTTP can connect with the endpoint and the `Authorization` header above. The Agent & MCP screen in the dashboard also gives you a block with your key already in it: paste it into your agent as a message and the agent sets itself up.

Then ask in plain words:

```text
"Draft a launch post, queue it for Tuesday 9am on LinkedIn and X,
 and tell me once it is live."
```

## Tools

Every connection gets the same 19 tools.

| Tool | Name |
|---|---|
| Which brand this is, and what clock it keeps | `get_workspace` |
| List channels | `list_channels` |
| What each network takes | `describe_networks` |
| Connect a channel | `connect_channel` |
| Stage a file you made | `stage_upload` |
| Upload media | `upload_media` |
| Describe an image | `describe_media` |
| Create and publish a post | `create_post` |
| Schedule a post | `schedule_post` |
| Schedule a batch of posts | `schedule_many` |
| Save a draft | `save_draft` |
| List drafts | `list_drafts` |
| Edit a draft or a queued post | `update_post` |
| Publish a draft | `publish_draft` |
| Get post status | `get_status` |
| List recent and queued posts | `list_queue` |
| Cancel a post | `cancel_post` |
| List delivery receipts | `list_events` |
| Health check | `health` |

A few things they are built to do:

- **Drafts wait for a person.** `save_draft` stores a post without queueing it, and nothing in a draft reaches a network until `publish_draft` runs.
- **A retry does not post twice.** `create_post`, `schedule_post` and `save_draft` take an idempotency key, and `schedule_many` takes a prefix, so repeating a call after a dropped connection returns what was already made instead of posting again.
- **Times are the brand's own.** A bare time such as `2026-11-03T09:00` is read in the workspace's time zone.
- **Receipts instead of assumptions.** `get_status` shows each channel's state with the live link or the error, and `list_events` is where a post that failed quietly shows up.

## Pricing

The MCP server is included on every plan, alongside the CLI and the REST API. Plans are priced per brand (one workspace each), with unlimited channels in every workspace.

| Plan | Monthly | Billed yearly | Brands | Scheduled posts a month |
|---|---|---|---|---|
| Starter | $15 | $10 a month | 1 | 200 |
| Startup | $29 | $23 a month | 3 | 400 |
| Plus | $67 | $54 a month | 10 | 1,000 |
| Pro | $99 | $79 a month | Unlimited | 2,500 |

Starter's first seven days are free: the card is taken when you subscribe, and nothing is charged until the week is out. Current prices are at https://postwharf.com/pricing.

## Links

- Website: https://postwharf.com
- Developer docs (REST API, MCP server and CLI): https://postwharf.com/developers
- MCP server guide, with what each network accepts: https://postwharf.com/mcp
- Support: support@postwharf.com
