# Cloud Storage Bridge Extension

**Version:** 1.0.2

> **⚠ Requires the `oauth-handler` extension.** Install it first (`workspace/oauth-handler.dsext`)
> and pipe `accessToken` from its *OAuth Get Token* node into any Cloud Storage Bridge node.
> See [Setup](#setup) below.

Upload, download, list, share, and create folders on **Google Drive**, **OneDrive**
(Microsoft Graph), or **Dropbox** from a single set of nodes. Authentication is
handled by the companion OAuth 2.0 Handler extension — install that first
and pipe its `accessToken` output into any Cloud Storage Bridge node.

## Setup

### Prerequisites

1. Install the **OAuth 2.0 Handler** extension (ships in this repo at
   `workspace/oauth-handler.dsext`).
2. Set up OAuth credentials with your provider:

#### Google Drive

1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project (or select existing), then enable the **Google Drive API**
3. Go to **APIs & Services → Credentials → OAuth consent screen**
   - Set to **External** (or **Internal** if your org owns the project)
   - Add the scopes your workflow needs (see "Scopes" below)
   - Add yourself as a test user — you can authorize without Google's $ verification
     if you stay in **Testing** publish status (up to 100 test users, free)
4. Create **OAuth 2.0 Client ID** (type "Web application"):
   - Add redirect URI: `http://127.0.0.1:8765/ext/oauth-handler/callback`
   - Save — copy the **Client ID** and **Client Secret**

#### OneDrive (Microsoft Graph)

1. Go to the [Azure Portal → App registrations](https://portal.azure.com/#blade/Microsoft_AAD_RegisteredApps/NewApplicationBlade)
2. Create a new registration:
   - **Redirect URI**: `http://127.0.0.1:8765/ext/oauth-handler/callback`
3. Go to **Certificates & secrets** → New client secret → copy the value
4. Go to **API permissions** → Add permission → Microsoft Graph → Delegated permissions:
   - `Files.ReadWrite`
   - `offline_access` (for refresh token)
   - `User.Read` (for `/me` endpoint)
5. Copy the **Application (client) ID** and **Directory (tenant) ID**

#### Dropbox

1. Go to the [Dropbox App Console](https://www.dropbox.com/developers/apps)
2. Click **Create app** → "Scoped access" → "Full Dropbox" (or "App folder" if you want to restrict to a subfolder)
3. Choose **OAuth 2** → **Permissions**:
   - `files.metadata.read` — list and resolve files/folders
   - `files.content.read` — download files
   - `files.content.write` — upload and create folders
   - `sharing.read` — create shared links
4. Click **Create app**, then copy the **App key** and **App secret**
5. Add redirect URI: `http://127.0.0.1:8765/ext/oauth-handler/callback` (under **OAuth 2** → **Redirect URIs**)
6. Set **Access token expiration** to "Short-lived" (the OAuth handler's refresh flow will extend it)

### Wire Up the OAuth Flow

Using the OAuth 2.0 Handler nodes in sequence:

```
1. OAuth Authorize  →  provider="google", clientId="…", scopes="https://www.googleapis.com/auth/drive.file", ...
   → opens browser, user grants permission, callback received at /ext/oauth-handler/callback
2. OAuth Get Token  →  provider="google", authorizationCode="<code from callback>", ...
   → outputs accessToken, refreshToken
3. Cloud Storage nodes →  accessToken="{{ steps.oauth_get_token_1.accessToken }}", ...
```

> **Tip:** For long-running workflows, pipe the `refreshToken` from step 2 into an
> `OAuth Refresh Token` node when the `accessToken` nears expiry, then use the new
> token in subsequent Cloud Storage nodes.

## Extension Settings (optional)

Configure on the Extensions page — values are used when a node leaves the field blank:

| Key | Type | Hint |
|---|---|---|
| `defaultProvider` | select (google_drive / onedrive / dropbox) | Pre-selects the provider on every node |
| `defaultAccessToken` | text (secret) | A static/bearer token fallback — useful for service accounts or testing without the OAuth flow |

## Nodes

All five nodes accept `provider` (select: `google_drive`, `onedrive`, `dropbox`)
and `accessToken` (secret text from the OAuth Get Token node).

### Upload File (`cloudstorage_upload_file`)
Uploads a local file to Google Drive, OneDrive, or Dropbox.

| Field | Type | Required | Notes |
|---|---|---|---|
| `provider` | select | ✓ | `google_drive`, `onedrive`, or `dropbox` |
| `accessToken` | text (secret) | ✓ | From OAuth Get Token |
| `localPath` | text | ✓ | Local file path |
| `remotePath` | text | | Destination path (e.g. `Documents/report.pdf`) |
| `parentId` | text | | Optional parent folder ID |

### Download File (`cloudstorage_download_file`)
Downloads a file from Google Drive, OneDrive, or Dropbox to local disk.

| Field | Type | Required | Notes |
|---|---|---|---|
| `provider` | select | ✓ | `google_drive`, `onedrive`, or `dropbox` |
| `accessToken` | text (secret) | ✓ | From OAuth Get Token |
| `fileId` | text | either fileId/remotePath | Cloud file ID |
| `remotePath` | text | either fileId/remotePath | Path in cloud storage |
| `localPath` | text | | Local output path (defaults to filename) |

### List Files (`cloudstorage_list_files`)
Lists files in a cloud storage folder.

| Field | Type | Required | Notes |
|---|---|---|---|
| `provider` | select | ✓ | `google_drive`, `onedrive`, or `dropbox` |
| `accessToken` | text (secret) | ✓ | From OAuth Get Token |
| `folderId` | text | either folderId/folderPath | Cloud folder ID or path |
| `folderPath` | text | either folderId/folderPath | Path in cloud storage (blank = root) |
| `pageSize` | number | | Max results (default: 50) |
| `query` | text | | Provider-specific filter query |

### Share File (`cloudstorage_share_file`)
Creates a shareable link or permission for a file.

| Field | Type | Required | Notes |
|---|---|---|---|
| `provider` | select | ✓ | `google_drive`, `onedrive`, or `dropbox` |
| `accessToken` | text (secret) | ✓ | From OAuth Get Token |
| `fileId` | text | ✓ | Cloud file ID or path |
| `role` | select | | `reader`, `writer`, `commenter` |

### Create Folder (`cloudstorage_create_folder`)
Creates a folder in Google Drive, OneDrive, or Dropbox.

| Field | Type | Required | Notes |
|---|---|---|---|
| `provider` | select | ✓ | `google_drive`, `onedrive`, or `dropbox` |
| `accessToken` | text (secret) | ✓ | From OAuth Get Token |
| `folderName` | text | ✓ | New folder name |
| `parentId` | text | | Parent folder ID or path |
| `parentPath` | text | | Parent folder path (e.g. `Documents/Projects`) |

## OAuth Scopes

### Google Drive
| Scope | What it allows |
|---|---|
| `https://www.googleapis.com/auth/drive.file` | Per-file access to files created/opened by this app (**recommended**) |
| `https://www.googleapis.com/auth/drive` | Full access to all files (needed for `share_file` with `type=anyone`) |

### Microsoft Graph (OneDrive)
| Scope | What it allows |
|---|---|
| `Files.ReadWrite` | Read and write a user's file access |
| `offline_access` | Access and refresh tokens (for long-lived access) |

### Dropbox
| Scope | What it allows |
|---|---|
| `files.metadata.read` | List and resolve files/folders |
| `files.content.read` | Download files |
| `files.content.write` | Upload and create folders |
| `sharing.read` | Create shared links |

## Limitations

- **OneDrive personal accounts only** (`/me/drive`). For SharePoint or OneDrive for Business shared libraries, use the site-relative endpoints — that requires a separate extension or a `siteId` parameter not yet supported here.
- **Google Drive paths are resolved** by walking the folder tree from root; deeply nested paths add extra API calls.
- **Dropbox file IDs** start with `id:` (e.g. `id:abc123`). When using `fileId` for download or share, include the `id:` prefix. Path-based lookups work for all Dropbox operations.
- **Large file uploads** (> 4 MB for OneDrive, > 5 MB for Google Drive multipart, > 150 MB for Dropbox) should ideally use resumable upload — this v1 implementation uses simple upload for all providers.
- **Token refresh is manual** — pipe the refresh token through an `OAuth Refresh Token` node when the access token expires.
- **The extension system has no `dependencies` manifest field** — the requirement on `oauth-handler` is documented in the manifest description and this README only. If you install this extension without the OAuth handler, nodes will fail at runtime with "Access token is required" rather than a clear install-time error.

## Dependencies

- `requests>=2.28.0` (installed in this extension's private `deps/` folder)
- The [OAuth 2.0 Handler extension](https://kilo.ai/docs) must be installed and configured separately.

## Contract (v1.0.2)

**Install:** Extensions → **Install from file** → choose `cloud-storage-bridge.dsext`.

**Permissions:**
- **network**
- **filesystem**
- **secrets**

No example workflow: needs cloud provider credentials.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/cloud-storage-bridge
python tools/deskstride_ext_cli.py pack extensions/cloud-storage-bridge -o cloud-storage-bridge.dsext
```
