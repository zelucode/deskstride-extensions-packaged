# Bitwarden Secrets Integration Extension

**Version:** 1.0.1

A Bitwarden vault integration for DeskStride that enables secure access to logins, secure notes, and other vault items in workflows.

## Features

- **Get Bitwarden Login**: Retrieve username, password, TOTP, and URIs from login items
- **Get Bitwarden Note**: Retrieve secure note content and custom fields
- **Get Bitwarden Field**: Retrieve specific fields from any item type (login, note, card, identity)

## Setup

### Install Bitwarden CLI

1. Download and install Bitwarden CLI from https://bitwarden.com/download/#cli
2. Verify installation: `bw --version`
3. Log in to Bitwarden: `bw login`
4. Unlock the vault: `bw unlock` (this will provide a session key)

### Extension Settings

Configure in the Extensions page:

- **Bitwarden CLI Path**: Path to Bitwarden CLI executable (default: `bw`)
- **Session Key**: Session key from `bw unlock` command (required, marked as secret)
- **Vault Timeout**: How long the vault stays unlocked in minutes (0 = never lock)

### Session Management

The session key is required for all Bitwarden CLI operations:

1. Run `bw unlock` in your terminal
2. Copy the session key (e.g., `BW_SESSION="..."`)
3. Paste it in the extension settings under "Session Key"
4. The session key will be used for all CLI commands

**Note:** Session keys expire based on your Bitwarden vault timeout settings. You'll need to update the session key when it expires.

## Nodes

### Get Bitwarden Login
Retrieve username, password, TOTP, and URIs from a Bitwarden login item.

**Inputs:**
- Item ID: Bitwarden item ID (preferred, faster lookup)
- Item Name: Bitwarden item name (slower, searches all items)

**Outputs:**
- success: Boolean indicating if retrieval succeeded
- itemId: The Bitwarden item ID
- itemName: The Bitwarden item name
- username: Username from the login item
- password: Password from the login item
- totp: TOTP secret/URI if configured
- uris: List of URIs associated with the login
- message: Status message

### Get Bitwarden Note
Retrieve secure note content and custom fields from Bitwarden.

**Inputs:**
- Item ID: Bitwarden item ID (preferred, faster lookup)
- Item Name: Bitwarden item name (slower, searches all items)

**Outputs:**
- success: Boolean indicating if retrieval succeeded
- itemId: The Bitwarden item ID
- itemName: The Bitwarden item name
- notes: The secure note content
- fields: Dictionary of custom fields
- message: Status message

### Get Bitwarden Field
Retrieve a specific field from any Bitwarden item type.

**Inputs:**
- Item ID: Bitwarden item ID (preferred, faster lookup)
- Item Name: Bitwarden item name (slower, searches all items)
- Field Name (required): Name of the field to retrieve

**Outputs:**
- success: Boolean indicating if retrieval succeeded
- itemId: The Bitwarden item ID
- itemName: The Bitwarden item name
- fieldName: The field name that was retrieved
- fieldValue: The value of the requested field
- message: Status message

## Supported Field Names

### Login Items (type 1)
- `username`: Login username
- `password`: Login password
- `totp`: TOTP secret/URI
- `uri`: Primary URI/URL

### Secure Notes (type 2)
- `notes`: Note content
- Custom field names as defined in the item

### Identity Items (type 4)
- Standard identity fields (title, firstName, lastName, etc.)
- Custom field names as defined in the item

### Card Items (type 3)
- Standard card fields (cardholderName, number, expMonth, etc.)
- Custom field names as defined in the item

## Use Cases

- **API Authentication**: Retrieve API keys and credentials for HTTP requests
- **Database Connections**: Get database credentials for connection strings
- **Service Accounts**: Access service account credentials for automation
- **Configuration Management**: Retrieve configuration values stored as secure notes
- **Password Rotation**: Automatically update credentials in external systems
- **Multi-environment Access**: Access different environment credentials (dev/staging/prod)

## Security Best Practices

- Session keys are stored as secrets in extension settings (encrypted)
- Never hardcode credentials in workflows
- Use Bitwarden's organization sharing for team credentials
- Rotate credentials regularly using Bitwarden's built-in tools
- Enable 2FA on your Bitwarden account
- Use separate Bitwarden items for different services

## Example Workflows

### API Request with Bitwarden Credentials
```
1. Get Bitwarden Login: itemName="API Service Account"
2. HTTP Request: 
   - URL: https://api.example.com/endpoint
   - Auth: Basic (username={{1.username}}, password={{1.password}})
```

### Database Connection
```
1. Get Bitwarden Field: itemName="Database", fieldName="password"
2. Connect to Database: 
   - Connection string: postgresql://user:{{1.fieldValue}}@db.example.com/db
```

### Configuration Retrieval
```
1. Get Bitwarden Note: itemName="App Configuration"
2. Parse JSON: input={{1.notes}}
3. Use configuration values in workflow
```

## Requirements

- Bitwarden CLI installed and accessible
- Bitwarden account with vault items
- Session key from `bw unlock` command
- No Python dependencies (uses Bitwarden CLI directly)

## Troubleshooting

### "Session key is required"
- Run `bw unlock` in your terminal
- Copy the session key (e.g., `BW_SESSION="..."`)
- Update the session key in extension settings

### "Bitwarden CLI failed"
- Verify Bitwarden CLI is installed: `bw --version`
- Check the CLI path in extension settings
- Ensure you're logged in: `bw login`
- Verify the session key is still valid

### "Item not found"
- Verify the item name or ID is correct
- Use Item ID instead of Item Name for faster lookup
- Check that the item exists in your vault
- Ensure you have access to the item (organization sharing)

### "Field not found"
- Verify the field name spelling (case-insensitive)
- Check the item type and available fields
- Use custom field names as defined in the item
- Try using the "Get Bitwarden Login" or "Get Bitwarden Note" nodes for full item data

## Notes

- Item ID lookups are faster than Item Name lookups
- Session keys expire based on Bitwarden vault timeout settings
- Passwords are returned in plain text for use in workflows
- TOTP secrets can be used with TOTP generators
- The extension uses Bitwarden CLI for all operations (no direct API)
- All secrets are handled securely via the extension's secret settings

## Limitations

- Requires Bitwarden CLI to be installed separately
- Session key management is manual (must be updated when expired)
- Read-only access (cannot create/update vault items)
- Depends on Bitwarden CLI being in PATH or specified path
- No support for Bitwarden Secrets Manager (personal vaults only)

## Contract (v1.0.1)

**Install:** Extensions → **Install from file** → choose `bitwarden-secrets.dsext`.

**Permissions:**
- **shell**
- **secrets**

No example workflow: needs Bitwarden CLI session key / vault credentials.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/bitwarden-secrets
python tools/deskstride_ext_cli.py pack extensions/bitwarden-secrets -o bitwarden-secrets.dsext
```
