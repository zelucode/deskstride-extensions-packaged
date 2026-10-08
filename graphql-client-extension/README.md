# GraphQL Client

**Version:** 1.0.1

Send GraphQL queries and mutations to any endpoint and inspect a server's schema. Uses only the standard library (no pip packages).
GraphQL errors, HTTP failures, non-JSON replies, unreachable hosts and timeouts each give a clear message.

## Nodes

| Node | What it does |
|---|---|
| **GraphQL Query** | Sends a query and returns `data`, `errors`, `hasErrors`, `status`, `durationMs`. Refuses a mutation or subscription so a write cannot be sent from the wrong node. |
| **GraphQL Mutation** | Same, for mutations (writes). |
| **GraphQL Introspect Schema** | Reads the server's schema: root type names, query/mutation field names, all type names, the full `schema` JSON, and optionally saves it to a file. |

**Fields (Query/Mutation):** Endpoint URL, the GraphQL document, Variables (a JSON object), Operation Name (when the document has several),
Extra Headers (JSON), Fail On GraphQL Errors (on by default), Timeout.

**Errors.** With *Fail On GraphQL Errors* on (default) a response containing `errors` stops the workflow with the server's messages.
Turn it off to continue and read `errors` / `hasErrors` yourself (useful for partial results). A server that answers HTTP 400 with a
GraphQL `errors` body is treated the same way. Anything that is not a GraphQL response (an HTML error page, plain JSON without
`data`/`errors`) is reported as a transport failure with the first part of the reply.

## Settings

- **Default Endpoint**: used when a node's Endpoint is blank.
- **Auth Token** (secret): sent as `Authorization: Bearer <token>`, or as-is when it already has a scheme (`Basic abc`). Skipped if you set an `Authorization` header yourself.
- **Default Headers (JSON)**: headers added to every request.
- **Timeout (seconds)**: default 30.

## Notes and limits

- Only `http://` and `https://` endpoints. Cloud metadata addresses (169.254.x.x) are refused by the SDK's network guard; local and LAN servers are fine.
- Subscriptions (websocket streams) are not supported.
- Replies larger than 20 MB are refused.
- Introspection must be enabled on the server; many production servers turn it off, and you then get that server's message.

## Contract (v1.0.0)

**Install:** Extensions -> **Install from file** -> choose `graphql-client-extension-1.0.0.dsext`.

**Permissions:**
- **network**
- **filesystem** (only to save the schema file when you set *Save Schema To*)

**Dependencies:** none.

**Develop:** `tests/extensions/test_graphql_client.py` runs the nodes against a local GraphQL-style server (success, partial errors, HTTP 400/502, non-JSON, timeout, introspection).
