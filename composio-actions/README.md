# Composio Actions — example extension

**Version:** 1.0.1

Adds a generic **Execute Composio Action** node covering
[Composio](https://composio.dev)'s catalog of 1000+ pre-authenticated SaaS
actions (Gmail, Slack, GitHub, Notion, ...), a **Check Composio Connection**
node for branching a workflow on connection status, and a **Composio
Connections** sidebar page for the per-user OAuth connect flow. Built for
`integration-lab/BACKLOG.md` item 1.

## Permissions

| Permission | Why |
|---|---|
| `network` | Calls Composio's cloud API to run tools and manage connections |
| `secrets` | Stores your Composio API key in the OS keychain via extension settings |

## Strategy: A (adopt the official SDK directly)

`LICENSE_NOTES.md`'s Composio entry (checked 2026-09-03, re-verified live
2026-09-04 via GitHub's license API — still **MIT**, unchanged) found
Composio to be MIT-licensed and already a Python-native library purpose-built
for exactly this: programmatic tool/action invocation across a large SaaS
catalog. `pipDependencies: ["composio==0.21.0"]` installs it into this
extension's private `deps/` folder (isolated from the app's own runtime —
the same mechanism `pdf-tools` uses for `pypdf`), and every node imports it
lazily inside its executor.

**Dependency footprint, checked before committing to this (per the concern
already flagged in `LICENSE_NOTES.md`):** `composio` 0.21.0 pulls in
`composio-client==1.43.0`, `openai>=2.48.0`, `pydantic>=2.11.9`,
`pysher>=1.0.8`, `jsonschema`, and a few smaller packages — real weight,
including a mandatory `openai` dependency this extension's own code never
imports (Composio's package bundles agent-framework integrations even
though this extension only uses its direct `tools.execute()` call). This is
an acceptable cost specifically *because* extension `pipDependencies` are
sandboxed into a private `deps/` folder per Ground rule about extension
isolation, not the shared core Python environment — it can't version-conflict
with anything else the app depends on. Requires Python ≥3.10; this app's
bundled runtime is 3.11+ (`scripts/setup-*`), so no compatibility gap.

## The one real scoping question: how does Composio's auth model map onto this app?

This is the actual work this backlog item called out (see item 1's "Effort"
note) — Composio's own auth model is **two-layered** and only the second
layer is something a workflow node can do:

1. **Auth Config** (per *app*, e.g. "GitHub OAuth") — created once, on
   Composio's own dashboard (`dashboard.composio.dev`). Nothing in this
   extension creates these; the SDK can (`composio.auth_configs.create(...)`)
   but that's an integration-*setup* action, not something a workflow should
   be doing at run time, so it's deliberately left as a one-time manual step
   outside this extension's scope.
2. **Connection** (per *user*, against an existing Auth Config) — this is
   what the **Composio Connections** page handles: `connected_accounts.initiate(...)`
   requests an OAuth connect link, `connected_accounts.list(...)` checks
   whether it went active. This app has no way to pop an OAuth browser
   window and catch the callback itself (no embedded browser / redirect
   listener in the extension runtime), so the page hands the user a plain
   URL to copy into their own browser rather than a clickable "Connect"
   button — the honest version of this flow given what's actually available,
   not a missing feature.

Once a user is connected (Step 2, either via this page or already connected
through Composio's own dashboard/another app), **Execute Composio Action**
just needs a `userId` + `toolSlug` + JSON `arguments` — the connection
lookup and credential handling all happen inside Composio's own service, not
in this extension.

## Try it

1. Sign up at [composio.dev](https://composio.dev), grab an API key from
   `dashboard.composio.dev/settings`, and create at least one **Auth Config**
   for an app you want to call (e.g. GitHub) — note its Auth Config ID.
2. Sidebar → **Extensions** → **Install from file...** → pick
   `composio-actions.dsext`.
3. Watch the streamed `pipDependencies` install log (real download — see
   the dependency footprint note above).
4. Set **Composio API Key** on the extension's settings card. Optionally set
   **Default User ID**.
5. Sidebar → **Extensions** group → **Composio Connections** → enter a User
   ID + your Auth Config ID → **Get Connect Link** → open the returned URL
   in a browser and approve → **Check Status** to confirm.
6. **Execute Composio Action** and **Check Composio Connection** appear in
   the palette's "composio" category.

No example workflow ships with this extension — every path needs a real
Composio API key, Auth Config, and connected account before it can run.

## What each node does

- **Execute Composio Action** (`composio_execute_action`): calls
  `client.tools.execute(tool_slug=..., user_id=..., arguments=...)` for
  *any* Composio tool slug — find one via
  [docs.composio.dev/toolkits](https://docs.composio.dev/toolkits) or
  Composio's own CLI (`composio search "send an email"`). Returns
  `successful` / `data` / a full `raw` JSON string; raises `RuntimeError`
  with Composio's own error message on a reported failure, since that
  message is almost always more specific than anything this wrapper could
  add.
- **Check Composio Connection** (`composio_check_connection`): returns
  `connected` (bool) + `connectionCount` for a user + Auth Config, meant to
  feed a Condition node so a workflow can skip/branch instead of letting
  Execute Composio Action fail outright on an unconnected account.

Both raise `NodeInputError` (a `user_mistake` in the Run Audit view) for a
missing API key, user id, tool slug, or Auth Config id — these are
almost-always setup mistakes, not transient failures.

## What this extension deliberately doesn't do

- **No per-service nodes.** The whole point of Strategy A here is that one
  generic node already covers the full catalog — hand-building a "Send
  Slack Message" / "Create GitHub Issue" node per service would just be
  re-deriving what `tools.execute()` already gives for free, and would
  drift out of sync with Composio's own catalog immediately.
- **No Auth Config creation.** See the auth-model section above.
- **No clickable in-app OAuth popup.** See the same section — the connect
  link is handed to the user as text, not opened automatically.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/composio-actions
python tools/deskstride_ext_cli.py scan extensions/composio-actions
python tools/deskstride_ext_cli.py pack extensions/composio-actions -o composio-actions.dsext
```
