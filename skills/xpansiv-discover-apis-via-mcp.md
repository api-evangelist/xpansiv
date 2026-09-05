---
name: xpansiv-discover-apis-via-mcp
description: Use Xpansiv's live remote MCP server to discover every Xpansiv API, its endpoints, its security schemes and its full OpenAPI document — then hand off to direct REST calls, because the MCP server cannot execute business operations.
api: Xpansiv Developer Portal MCP Server
endpoint: https://developer.xpansiv.com/mcp
generated: '2026-09-04'
method: generated
source: mcp/xpansiv-mcp-tools-list.json (live tools/list, HTTP 200, 2026-09-04) + mcp/xpansiv-tool-crosswalk.yml
operations:
  - list-apis
  - get-endpoints
  - get-endpoint-info
  - get-security-schemes
  - get-full-spec-document
  - search
  - whoami
---

# Discover Xpansiv's APIs through MCP

Xpansiv runs a remote MCP server at **`https://developer.xpansiv.com/mcp`**. As of
2026-09-04 `tools/list` answers anonymously and returns seven tools with full input
schemas.

**Know what this server is before you plan around it.** It is a *documentation* server. Its
tools read Xpansiv's OpenAPI descriptions and search its docs. **Not one of them calls a
Xpansiv business API** — you cannot retire a certificate, move a holding, submit a meter
reading or place an order through it. Zero of the seven tools bind to any of the 110
operations in Xpansiv's eleven published contracts. Use it to *understand* the surface,
then call the REST APIs directly with credentials Xpansiv issues you.

## Connecting

```
POST https://developer.xpansiv.com/mcp
Content-Type: application/json
Accept: application/json, text/event-stream

{"jsonrpc":"2.0","id":1,"method":"tools/list"}
```

Responses are framed as SSE. RFC 9728 protected-resource metadata is published at
`https://developer.xpansiv.com/.well-known/oauth-protected-resource/mcp`
(scopes `openid`, `profile`, `email`, `offline_access`), but discovery answered without a
token. Authentication appears to scope `whoami` and account-bound content.

## A discovery pass

1. **`list-apis`** — optional `name` filter. Enumerates the APIs the portal describes:
   Xpansiv Connect, Xpansiv Managed Solutions, the NAR and TIGRS registry client APIs, the
   six Optimal Outcomes services and the APX Power Markets file registry.
2. **`get-endpoints`** — requires `name`. All endpoints for one API.
3. **`get-endpoint-info`** — requires `name`, `path` and `method`. Parameters, security and
   examples for one operation. This is the MCP equivalent of reading an OpenAPI operation
   object, and it is where to look before writing a call.
4. **`get-security-schemes`** — requires `name`. Which credential the API expects. Do this
   per API: Xpansiv's families do **not** share an authorization server (Connect and Data
   use `auth.xpansiv.com`; NAR, TIGR and Power Markets use `apxjwtauthprod.apx.com`;
   Managed Solutions uses a long-lived account API key).
5. **`get-full-spec-document`** — requires `name`. The whole OpenAPI document.
6. **`search`** — requires `query`, a single word or an exact documented phrase. Reaches
   the prose the specs do not carry: the FIX 4.4 marketplace rules of engagement, the NCSV
   time-series format, the APXScheduleAPI XSD and the Xpansiv Data SDK guide.

## Where MCP stops

Two Xpansiv surfaces are not described by any OpenAPI document and so are not reachable
through `get-endpoints` at all:

- **Xpansiv Data** (`https://api.data.xpansiv.com`) — documented in prose only; use
  `search` for it, or the `xpansiv-data` Python SDK on PyPI.
- **Xpansiv Marketplace** — order entry and real-time market data for CBL run over
  **FIX 4.4**, a session protocol with no HTTP surface. `search` finds the rules of
  engagement; nothing can call it as a tool.

## After discovery

Everything executable lives behind credentials Xpansiv issues to a contracted client. See
`skills/xpansiv-retire-environmental-commodity.md`,
`skills/xpansiv-transfer-and-reverse.md` and `skills/xpansiv-submit-iso-schedule.md` for the
three flows and the hazards — no idempotency keys anywhere, irreversible retirements, and
asynchronous validation on file uploads.
