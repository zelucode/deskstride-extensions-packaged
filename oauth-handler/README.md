# OAuth 2.0 Handler Extension

**Version:** 1.1.1

An OAuth 2.0 flow management extension for DeskStride that provides token acquisition, refresh, and authenticated request wrappers for cloud service integrations.

## Features

- **OAuth Authorize**: Initiate OAuth 2.0 authorization flow and return authorization URL
- **OAuth Get Token**: Exchange authorization code for access token
- **OAuth Refresh Token**: Refresh access token using refresh token
- **OAuth Authenticated Request**: Make authenticated HTTP requests with OAuth tokens
- **HTTP Callback Handler**: Handle OAuth callbacks from authorization servers

## Setup

### Extension Settings

Configure in the Extensions page:

- **OAuth Callback URL**: OAuth callback URL for authorization flow (e.g., `http://127.0.0.1:8765/ext/oauth-handler/callback`)
- **Token Storage**: How to store OAuth tokens (`memory` = lost on restart, `extension_state` = persisted)

### HTTP Routes

This extension provides an HTTP route for OAuth callbacks:

1. Go to Settings → Server (Local API)
2. Enable the local API server
3. On the Extensions card, enable **Allow HTTP routes**
4. The callback endpoint will be available at:
   ```
   http://127.0.0.1:8765/ext/oauth-handler/callback
   ```

### Provider Setup

#### Google
1. Go to [Google Cloud Console](https://console.cloud.google.com/)
2. Create a new project or select existing
3. Enable required APIs (e.g., Google Drive API)
4. Create OAuth 2.0 credentials (Web application)
5. Add the callback URL to authorized redirect URIs
6. Copy Client ID and Client Secret

#### GitHub
1. Go to [GitHub Developer Settings](https://github.com/settings/developers)
2. Register a new OAuth application
3. Set the callback URL
4. Copy Client ID and Client Secret

#### Slack
1. Go to [Slack API](https://api.slack.com/apps)
2. Create a new app
3. Enable OAuth & Permissions
4. Add redirect URL
5. Copy Client ID and Client Secret

#### Salesforce
1. Go to [Salesforce Setup](https://login.salesforce.com/setup)
2. Create a Connected App
3. Enable OAuth settings
4. Add callback URL
5. Copy Consumer Key and Consumer Secret

#### HubSpot
1. Go to [HubSpot Developer Portal](https://developers.hubspot.com/)
2. Create an app
3. Configure OAuth
4. Add redirect URL
5. Copy Client ID and Client Secret

## Nodes

### OAuth Authorize
Initiate OAuth 2.0 authorization flow and return authorization URL for user to complete in browser.

**Inputs:**
- Provider (required): OAuth provider (google, github, slack, salesforce, hubspot, generic)
- Client ID (required): OAuth client ID from provider
- Client Secret: OAuth client secret from provider
- Scopes: OAuth scopes (space-separated)
- Redirect URI: OAuth redirect URI (uses extension setting if blank)
- State: OAuth state parameter (auto-generated if blank)
- Authorization URL: Custom authorization URL (for generic provider)
- Token URL: Custom token URL (for generic provider)

**Outputs:**
- success: Boolean indicating if authorization initiated
- authorizationUrl: URL to open in browser for user authorization
- state: OAuth state parameter
- provider: The OAuth provider
- redirectUri: The redirect URI used
- message: Status message

### OAuth Get Token
Exchange OAuth authorization code for access token to complete the OAuth flow.

**Inputs:**
- Provider (required): OAuth provider
- Client ID: OAuth client ID from provider
- Client Secret: OAuth client secret from provider
- Authorization Code (required): OAuth authorization code from callback
- Redirect URI: OAuth redirect URI (uses extension setting if blank)
- Token URL: Custom token URL (for generic provider)

**Outputs:**
- success: Boolean indicating if token obtained
- accessToken: OAuth access token
- refreshToken: OAuth refresh token
- expiresIn: Token expiration time in seconds
- tokenType: Token type (usually Bearer)
- provider: The OAuth provider
- message: Status message

### OAuth Refresh Token
Refresh an OAuth 2.0 access token using the refresh token to maintain authenticated sessions.

**Inputs:**
- Provider (required): OAuth provider
- Client ID: OAuth client ID from provider
- Client Secret: OAuth client secret from provider
- Refresh Token (required): OAuth refresh token
- Token URL: Custom token URL (for generic provider)

**Outputs:**
- success: Boolean indicating if token refreshed
- accessToken: New OAuth access token
- refreshToken: New refresh token (or original if not rotated)
- expiresIn: Token expiration time in seconds
- tokenType: Token type
- provider: The OAuth provider
- message: Status message

### OAuth Authenticated Request
Make an authenticated HTTP request using OAuth access token. Handles authorization headers automatically.

**Inputs:**
- URL (required): The URL to make the authenticated request to
- Method: HTTP method (GET, POST, PUT, DELETE, PATCH)
- Access Token (required): OAuth access token
- Token Type: Token type (default: Bearer)
- Headers: Additional headers as JSON object
- Body: Request body as JSON or plain text

**Outputs:**
- success: Boolean indicating if request completed
- statusCode: HTTP status code
- response: Response data (JSON or text)
- headers: Response headers
- url: The requested URL
- method: The HTTP method used
- message: Status message

## OAuth Flow

### Standard OAuth 2.0 Authorization Code Flow

1. **OAuth Authorize**: Get authorization URL
   - User receives authorization URL
   - Opens URL in browser
   - Logs into provider and grants permissions

2. **Callback Handling**: Provider redirects to callback URL
   - Extension's HTTP route receives callback
   - Extracts authorization code and state

3. **OAuth Get Token**: Exchange code for tokens
   - Use authorization code from callback
   - Obtain access token and refresh token
   - Store tokens securely

4. **OAuth Authenticated Request**: Use access token
   - Make authenticated API requests
   - Access provider's services

5. **OAuth Refresh Token**: Maintain session
   - When access token expires
   - Use refresh token to get new access token
   - Continue without user interaction

## Use Cases

- **Google Drive Integration**: Access Google Drive files and folders
- **GitHub Automation**: Manage repositories, issues, and pull requests
- **Slack Integration**: Send messages and manage channels
- **Salesforce Integration**: Access CRM data and automate workflows
- **HubSpot Integration**: Manage contacts, deals, and marketing automation
- **Custom OAuth**: Integrate with any OAuth 2.0 compatible service

## Example Workflows

### Google Drive OAuth Flow
```
1. OAuth Authorize: 
   - provider="google"
   - clientId="{{google_client_id}}"
   - scopes="https://www.googleapis.com/auth/drive.readonly"
2. [User opens authorization URL and grants permissions]
3. OAuth Get Token:
   - provider="google"
   - authorizationCode="{{callback_code}}"
4. OAuth Authenticated Request:
   - url="https://www.googleapis.com/drive/v3/files"
   - accessToken="{{3.accessToken}}"
```

### Slack OAuth with Token Refresh
```
1. OAuth Authorize: provider="slack", clientId="{{slack_client_id}}"
2. [User authorizes Slack app]
3. OAuth Get Token: provider="slack", authorizationCode="{{callback_code}}"
4. OAuth Authenticated Request: url="https://slack.com/api/conversations.list", accessToken="{{3.accessToken}}"
5. If token expires:
   - OAuth Refresh Token: provider="slack", refreshToken="{{3.refreshToken}}"
```

## Common OAuth Scopes

### Google
- `https://www.googleapis.com/auth/drive.readonly` - Read Google Drive files
- `https://www.googleapis.com/auth/drive` - Full Google Drive access
- `https://www.googleapis.com/auth/gmail.readonly` - Read Gmail
- `https://www.googleapis.com/auth/calendar` - Google Calendar access

### GitHub
- `repo` - Full repository access
- `user` - User profile access
- `user:email` - User email access
- `repo:status` - Commit status access

### Slack
- `channels:read` - Read channels
- `channels:write` - Write to channels
- `chat:write` - Send messages
- `files:write` - Upload files

### Salesforce
- `api` - Full API access
- `refresh_token` - Offline access
- `full` - Full access

## Token Storage

### Memory Storage (Default)
- Tokens stored in memory only
- Lost when extension is unloaded or app restarts
- Suitable for testing and short-lived sessions
- Requires re-authorization on restart

### Extension State Storage
- Tokens persisted in node state
- Survives extension reloads and app restarts
- Suitable for production use
- More secure than memory storage

## Security Best Practices

- **Secret Storage**: Store client secrets as secrets in extension settings
- **HTTPS Only**: Always use HTTPS for OAuth endpoints
- **State Parameter**: Always use state parameter to prevent CSRF attacks
- **Token Security**: Never log or expose access tokens
- **Scope Limitation**: Request minimum required scopes
- **Token Refresh**: Implement token refresh before expiration
- **Revocation**: Implement token revocation when possible

## Requirements

- requests==2.31.0 (included in extension)
- requests-oauthlib==1.3.1 (included in extension)
- Local API server enabled for HTTP routes
- OAuth client credentials from providers

## Troubleshooting

### "Redirect URI is required"
- Configure the callback URL in extension settings
- Ensure the callback URL matches your OAuth app configuration
- Enable HTTP routes in extension settings

### "Authorization code is required"
- Complete the OAuth flow by opening the authorization URL
- Copy the authorization code from the callback
- Use the code in the OAuth Get Token node

### "Failed to refresh token"
- Check that the refresh token is still valid
- Verify client credentials are correct
- Some providers may not issue refresh tokens

### "Invalid grant" error
- Authorization code may have expired (usually 10 minutes)
- Refresh token may have been revoked
- Check provider's OAuth documentation

## Notes

- Extension supports authorization code flow (most secure)
- Built-in configurations for major providers (Google, GitHub, Slack, Salesforce, HubSpot)
- Generic provider support for custom OAuth implementations
- HTTP callback handler for OAuth provider redirects
- Token storage options for different security requirements
- State parameter automatically generated for CSRF protection
- Tokens can be used across multiple authenticated requests

## Contract (v1.1.1)

**Install:** Extensions → **Install from file** → choose `oauth-handler.dsext`.

**Permissions:**
- **network**
- **httpRoutes**

No example workflow: needs OAuth client credentials and callback flow.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/oauth-handler
python tools/deskstride_ext_cli.py pack extensions/oauth-handler -o oauth-handler.dsext
```

## Sidebar page

Sidebar **OAuth Console**: build authorize URLs, exchange codes, view/clear the page token session.
