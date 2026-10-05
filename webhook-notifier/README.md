# Webhook Notifier (example extension)

A real, working extension — adds a **Notify Webhook** node that POSTs a
workflow event to a configured webhook URL, plus a `POST /receive` +
`GET /status` HTTP route pair so you can test the whole loop without
depending on any real third-party service. Meant as a worked example of
Phase 3 (settings, incl. secrets, and HTTP routes) to read alongside
[docs/guide/creating-extensions.md](../../../docs/guide/creating-extensions.md)
and [pdf-tools](../pdf-tools/README.md) (Phase 1/2's example), not as a
production notification service.

## Try it — send to a route of your own

1. Sidebar → **Extensions** → **Install from file...**
2. Pick `../webhook-notifier.dsext` (the pre-built package, sibling to this
   directory) — or zip this directory yourself: `manifest.json`, `nodes/`,
   and `handlers/` go at the zip's root, no wrapping folder.
3. Confirm the trust warning. No `pipDependencies` — this extension only
   uses `httpx`, already bundled with the app.
4. On the extension's card, set **Webhook URL** to anywhere that accepts a
   POST and echoes/logs it (e.g. a URL from <https://webhook.site>, or your
   own test endpoint). It's a secret field, so it's stored in the OS
   keychain, not the database.
5. Drop **Notify Webhook** (palette → "http" category) into a workflow, set
   **Message**, and run it. `steps.<ref>.status`/`.body` hold the response.

## Try it — fully self-contained (no external service at all)

The extension can send to *itself*:

1. Settings → Server → **Enable Local API**, then restart the app (the
   server only starts at launch — see
   [docs/features/local-http-api.md](../../../docs/features/local-http-api.md)).
2. Settings → Server → copy the **API key**.
3. Back on the Extensions page, for this extension: turn on **Allow HTTP
   routes**, then set:
   - **Webhook URL** → `http://127.0.0.1:8765/ext/webhook-notifier/receive`
   - **Bearer Token** → the API key you copied
4. Run **Notify Webhook** again. It now POSTs to this same extension's own
   `receive` handler, which echoes the message back with a timestamp —
   verifiable end to end (settings → secrets → node → local API auth →
   HTTP route → handler → response) without any real webhook target.
5. `curl http://127.0.0.1:8765/ext/webhook-notifier/status -H "Authorization: Bearer <key>"`
   is a side-effect-free way to check the route/auth wiring on its own.
   Turn "Allow HTTP routes" off and the same call returns 403, not 404 — the
   route still exists, it's just gated.

## What each file does

- **`nodes/notify_webhook.py`**: the **Notify Webhook** node. Resolves its
  target URL from the node's own field, falling back to
  `ctx.ext["webhookUrl"]` (same fallback pattern pdf-tools uses for
  `defaultOutputDir`); adds `Authorization: Bearer <ext.authToken>` when
  set; retries up to `ext.retryCount` times; reuses core's
  `nodes._net_guard.assert_safe_url` outbound check.
- **`handlers/_util.py`**: shared `now_iso()` helper, not listed in
  `httpRoutes` — imported by both handlers via a relative import
  (`from ._util import now_iso`), the same sibling-import mechanism
  `nodes/_pdf.py` demonstrates for `nodeModules`, exercised here under
  `handlers/` instead.
- **`handlers/receive.py`**: `POST /ext/webhook-notifier/receive` —
  echoes back whatever body it got, plus the resolved (non-secret)
  settings visible to a handler.
- **`handlers/status.py`**: `GET /ext/webhook-notifier/status` —
  trivial health check, no state, safe to hit repeatedly.

## Settings (Extensions page)

| Key | Type | Secret | Notes |
|---|---|---|---|
| `webhookUrl` | text | yes | Default target for Notify Webhook when the node's own URL field is blank. |
| `authToken` | text | yes | Sent as `Authorization: Bearer <token>`. |
| `retryCount` | number | no | Extra POST attempts after the first, on failure. |
| `verboseLogging` | boolean | no | Logs each attempt's outcome to the run's stderr. |
| `logLevel` | select | no | `info` / `warn` / `error` — cosmetic prefix on verbose log lines, also echoed by `receive`. |
