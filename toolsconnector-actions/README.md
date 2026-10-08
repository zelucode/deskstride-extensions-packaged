# ToolsConnector Actions — example extension

**Version:** 1.1.1

Adds a generic **Execute ToolsConnector Action** node covering [ToolsConnector](https://toolsconnector.github.io/)'s catalog of 77+ connectors and 1,578+ actions (Gmail, Slack, GitHub, Stripe, Notion, Google Workspace, 14 AWS services, and more), plus a **List ToolsConnector Connectors** node for discovery, and a **ToolsConnector Browser** sidebar page. Built for `integration-lab/BACKLOG.md` item 16.

## Permissions

| Permission | Why |
|---|---|
| `network` | Calls the SaaS APIs for the connectors you configure |
| `secrets` | Stores connector API keys/tokens in the OS keychain |

## Key Features & Advantages

### 🚀 **Unified API Surface**
- **One generic node** covers 1,578+ actions across 77+ connectors — no per-service node proliferation
- **Consistent interface** across all services: connector + action + JSON arguments
- **Type-safe operations** with Pydantic models for all inputs and outputs
- **Auto-generated schemas** for OpenAI, Anthropic, and Gemini function calling

### 🔒 **Privacy-First BYOK Model**
- **Bring Your Own Keys** — API keys never leave your stack
- **No hosted service dependency** — runs entirely in-process
- **No external auth flows** — paste ready-to-use tokens directly
- **Full data sovereignty** — credentials stored only in extension settings

### 📦 **Minimal Dependency Footprint**
- **Core requires only 3 packages**: pydantic, httpx, docstring-parser
- **Selective connector installation** — only load what you need
- **Private deps/ isolation** — no version conflicts with core app
- **Fast install** — lightweight compared to full SDK bundles

### 🤖 **AI-Native Design**
- **Built-in MCP server support** — expose connectors to Claude Desktop, Cursor, Windsurf
- **Dual-use architecture** — works identically for traditional apps and AI agents
- **Schema generation** — automatic function-calling schemas for multiple LLM providers
- **Agent-ready** — designed from the ground up for AI/LLM integration

### 🛠 **Developer-Friendly**
- **Simple credential management** — extension settings UI for common connectors
- **Discovery node** — list available connectors and their configuration status
- **Easy extensibility** — add new connectors by updating credential map
- **Clear error messages** — setup mistakes vs. runtime failures distinguished

### 🌐 **Broad Coverage**
- **77+ connectors** across 20 categories: communication, social, databases, DevOps, CRM, AI/ML, AWS infrastructure
- **1,578+ typed actions** — from simple CRUD to complex workflows
- **Popular services included**: Gmail, Slack, GitHub, Stripe, Notion, Google Workspace, 14 AWS services
- **Regular updates** — new connectors and actions added frequently

### ⚡ **Performance & Reliability**
- **In-process execution** — no network latency from external service calls
- **Async-first, sync-friendly** — every action supports both execution modes
- **Resilient HTTP** — built on httpx with configurable retry policies
- **Type safety** — compile-time guarantees with Pydantic V2

## Strategy: A (adopt the official SDK directly)

`LICENSE_NOTES.md`'s ToolsConnector entry (checked 2026-09-05 via direct LICENSE file fetch from GitHub) found ToolsConnector to be Apache-2.0 licensed and already a Python-native library designed for exactly this: programmatic API integration across a large SaaS catalog. `pipDependencies: ["toolsconnector==0.3.25"]` installs it into this extension's private `deps/` folder (isolated from the app's own runtime — the same mechanism `pdf-tools` uses for `pypdf`), and every node imports it lazily inside its executor.

**Dependency footprint:** ToolsConnector has a minimal core footprint — only `pydantic`, `httpx`, and `docstring-parser` are required. Individual connectors can be installed selectively (`pip install "toolsconnector[gmail,slack,github]"`) to keep the extension lightweight. This is an acceptable cost specifically *because* extension `pipDependencies` are sandboxed into a private `deps/` folder per the extension isolation model, not the shared core Python environment — it can't version-conflict with anything else the app depends on.

## The ToolsConnector model: BYOK (Bring Your Own Keys)

ToolsConnector's auth model is simpler than Composio's (item 1) — there's no hosted service or OAuth dance. Each connector uses a straightforward API key or token that you provide directly:

- **Gmail**: OAuth2 access token or API key
- **Slack**: Bot token (xoxb-...)
- **GitHub**: Personal access token
- **Stripe**: API key
- **Notion**: Integration token
- ...and similar patterns for other connectors

This extension's **Settings** card has fields for the most common connectors. You only need to fill in the ones you actually use — the extension gracefully handles missing credentials for connectors you don't need.

**Why this matters:** Unlike Composio (which requires Auth Config setup on their dashboard and a two-layer connection flow), ToolsConnector runs entirely in-process with your own keys. Nothing leaves your stack, and there's no external dependency on a hosted service. This makes it ideal for:
- Privacy-sensitive deployments where you can't use hosted services
- Environments with strict network policies
- Users who prefer direct API control over abstracted service layers

## Try it

1. Get API keys/tokens for the services you want to use:
   - **Gmail**: Google Cloud Console OAuth2 credentials or Gmail API key
   - **Slack**: Create a Slack app and get a bot token from https://api.slack.com/apps
   - **GitHub**: Personal access token from https://github.com/settings/tokens
   - **Stripe**: API key from https://dashboard.stripe.com/apikeys
   - **Notion**: Integration token from https://www.notion.so/my-integrations

2. Sidebar → **Extensions** → **Install from file...** → pick `toolsconnector-actions.dsext`.

3. Watch the streamed `pipDependencies` install log (ToolsConnector + its minimal core dependencies).

4. Set your API keys on the extension's settings card — only fill in the connectors you actually plan to use.

5. **Execute ToolsConnector Action** and **List ToolsConnector Connectors** appear in the palette's "toolsconnector" category.
6. Sidebar → **Extensions** group → **ToolsConnector Browser** — see which connectors have keys, list actions, or try one action with JSON arguments (calls the real API).

No example workflow ships with this extension — every useful path needs real SaaS credentials before it can run.

## What each node does

- **Execute ToolsConnector Action** (`toolsconnector_execute_action`): calls any ToolsConnector action given a connector name (e.g. `gmail`, `slack`), action name (e.g. `send_message`, `post_message`), and JSON arguments. Returns `successful` / `data` / a full `raw` JSON string, plus the connector and action used. Raises `NodeInputError` for missing connector/action or invalid JSON, and `RuntimeError` with ToolsConnector's own error message on execution failures.
- **List ToolsConnector Connectors** (`toolsconnector_list_connectors`): returns which connectors are available, which have credentials configured, and documentation links. Useful for discovery and debugging.

Both nodes raise `NodeInputError` (a `user_mistake` in the Run Audit view) for missing configuration (no API keys set) or invalid input — these are almost-always setup mistakes, not transient failures.

## Examples

### Send a Gmail message
```
Connector: gmail
Action: send_message
Arguments JSON: {"to": "user@example.com", "subject": "Hello from DeskStride", "body": "This is an automated message."}
```

### Post a Slack message
```
Connector: slack  
Action: post_message
Arguments JSON: {"channel": "#general", "text": "Hello from DeskStride!"}
```

### Create a GitHub issue
```
Connector: github
Action: create_issue  
Arguments JSON: {"repo": "owner/repo", "title": "Bug report", "body": "Found an issue"}
```

## What this extension deliberately doesn't do

- **No per-service nodes.** The whole point of Strategy A here is that one generic node already covers the full catalog — hand-building a "Send Gmail Message" / "Post Slack Message" node per service would just be re-deriving what the ToolsConnector SDK already gives for free.
- **No OAuth dance handling.** ToolsConnector expects you to provide ready-to-use tokens (OAuth2 access tokens, API keys, etc.) — this extension doesn't implement OAuth flows itself. For services that require OAuth, you'll need to obtain the access token through their official OAuth flow and paste it into the extension settings.
- **No hosted service dependency.** Unlike Composio, ToolsConnector runs entirely in-process with your own keys — there's no external service to depend on or pay for.

## Extending the connector list

The current implementation includes credential fields for the most common connectors (Gmail, Slack, GitHub, Stripe, Notion). To add more:

1. Add the connector to `CONNECTOR_CREDENTIAL_MAP` in `nodes/_toolsconnector.py`
2. Add a corresponding settings field in `manifest.json`
3. Consult the [ToolsConnector documentation](https://toolsconnector.github.io/) for the connector's credential format

The extension is designed to make this straightforward — the credential mapping is centralized in one place, and adding a new connector is just a few lines of configuration.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/toolsconnector-actions
python tools/deskstride_ext_cli.py scan extensions/toolsconnector-actions
python tools/deskstride_ext_cli.py pack extensions/toolsconnector-actions -o toolsconnector-actions.dsext
```

## Comparison with Composio (item 1)

| Aspect | ToolsConnector (this extension) | Composio (item 1) |
|--------|--------------------------------|-------------------|
| **Architecture** | In-process Python library | Hosted service |
| **Auth model** | BYOK (bring your own keys) | Two-layer (Auth Config + Connection) |
| **Dependency footprint** | Minimal (pydantic, httpx, docstring-parser) | Larger (includes openai, pydantic, pysher) |
| **External dependencies** | None (runs locally) | Requires Composio service access |
| **Connector count** | 77+ connectors, 1,578+ actions | 1000+ actions across many services |
| **Setup complexity** | Low (paste API keys) | Medium (dashboard setup + OAuth flow) |
| **Privacy** | Keys never leave your stack | Credentials stored in Composio service |
| **AI integration** | Built-in MCP server support | Agent-focused design |

Both are high-leverage integrations that avoid per-service node proliferation. ToolsConnector is ideal when you want direct API control and privacy, while Composio excels when you prefer a managed service with pre-built OAuth flows.
