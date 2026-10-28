# Verifox MCP Server

[![smithery badge](https://smithery.ai/badge/verifox-ai/email-verifier)](https://smithery.ai/servers/verifox-ai/email-verifier)

MCP (Model Context Protocol) server that gives Claude and other AI assistants the **Verifox** API — **verify email deliverability** and **find business emails**, both single and in bulk.

Requires **Node.js 18 or newer** (the client runs it with `npx`).

## Quick start (published package)

Add to your Claude Desktop config (`~/Library/Application Support/Claude/claude_desktop_config.json`) or Claude Code MCP config — no install needed, `npx` fetches it:

```json
{
  "mcpServers": {
    "verifox": {
      "command": "npx",
      "args": ["-y", "verifox-mcp"],
      "env": {
        "VERIFOX_API_KEY": "foxkey_your_key_here"
      }
    }
  }
}
```

Get a free API key (1,000 credits) at **https://verifox.ai/dashboard/api**. Restart Claude Desktop and ask it to "verify these emails".

Config paths and add-commands for Claude Code, Claude Desktop, ChatGPT, Cursor, Gemini CLI, Kiro and Codex: **https://verifox.ai/email-verification-mcp**

## Tools

| Tool            | Does                                                                                    | Credits      |
| --------------- | --------------------------------------------------------------------------------------- | ------------ |
| `verify_email`  | Verify ONE email — SMTP deliverability, catch-all, disposable, role, free + 0-100 score | 1 / email    |
| `verify_emails` | Bulk-verify a LIST (1–100,000 emails) — submits, waits, returns one result per email    | 1 / email    |
| `find_email`    | Find ONE business email from name + company domain (best match or not_found)            | 10 / find    |
| `find_emails`   | Bulk-find emails for a LIST of up to 1,000 contacts (name + domain each)                | 10 / contact |
| `get_credits`   | Check the account's remaining Fox Credit balance                                        | free         |
| `get_job`       | Poll a long-running `verify_emails` / `find_email` / `find_emails` job for results      | —            |

Verify and find are async for large lists: the bulk tools wait ~55s inline, and hand back a `jobId` to finish via `get_job` if a job runs longer. Every verification costs 1 credit, including a repeat answered from the 24-hour result cache.

## Usage examples

In Claude Desktop (or any MCP client), just ask:

- "Is john@stripe.com deliverable?" → `verify_email`
- "Verify these emails: admin@stripe.com, test@mailinator.com, john.doe@microsoft.com" → `verify_emails`
- "Clean this lead list and keep only the safe addresses" → `verify_emails`
- "Find the email for Patrick Collison at stripe.com" → `find_email`
- "Find emails for these 20 people and their company domains" → `find_emails`
- "How many Verifox credits do I have left?" → `get_credits`
- "Check on that verification job" → `get_job`

## Hosted remote endpoint

No local install: point a remote-capable MCP client at

```
https://mcp.verifox.ai/mcp
```

(streamable HTTP) and send your key in the `x-api-key` header.

## Local development

Run straight from source with Bun (no build):

```bash
bun install
```

```json
{
  "mcpServers": {
    "verifox": {
      "command": "bun",
      "args": ["run", "/FULL/PATH/TO/verifox-mcp/src/index.ts"],
      "env": {
        "VERIFOX_API_URL": "https://api.verifox.ai",
        "VERIFOX_API_KEY": "foxkey_your_key_here"
      }
    }
  }
}
```

`VERIFOX_API_URL` defaults to `https://api.verifox.ai` when unset. For local
development against the API on port 8001, set it to `http://localhost:8001`.
Restart your client and the tools appear in its tool list.

## Remote (HTTP) mode

The same server also runs as a **streamable-HTTP** MCP endpoint (for Smithery,
claude.ai connectors, and other remote MCP clients). Set `MCP_TRANSPORT=http`
(or a `PORT`) and it listens on `POST /mcp` with a `GET /health` check:

```bash
MCP_TRANSPORT=http PORT=8080 node dist/index.js
# or: docker build -t verifox-mcp . && docker run -p 8080:8080 verifox-mcp
```

In HTTP mode the Verifox API key is read **per request** from the
`Authorization: Bearer <foxkey>` header (also accepts `x-api-key` or `?api_key=`)
— so one hosted endpoint serves many users, each on their own credits.

## Environment Variables

| Variable          | Default                  | Description                                        |
| ----------------- | ------------------------ | -------------------------------------------------- |
| `VERIFOX_API_URL` | `https://api.verifox.ai` | Verifox API base URL                               |
| `VERIFOX_API_KEY` | (empty)                  | Per-user API key (`foxkey_*`), sent as `x-api-key` |

Generate a `foxkey_*` key from the Verifox dashboard (API Keys). The key
authenticates every tool call and is the account whose Fox Credits are charged
(verify costs 1 credit per email, find 10 per contact; `get_credits` is free).
