# Discord Integration Extension

**Version:** 1.0.2

A Discord integration for DeskStride that enables sending messages, posting to channels, and uploading files via webhook or bot token.

## Features

- **Send Discord Message**: Send messages via Discord webhook (simple, no bot required)
- **Post to Discord Channel**: Post messages to channels using bot token
- **Upload Discord File**: Upload files (images, documents, etc.) via webhook or bot

## Setup

### Webhook Mode (Recommended - Simple)

1. Go to your Discord server settings → Integrations → Webhooks
2. Create a new webhook
3. Copy the webhook URL
4. Paste the webhook URL in the extension settings or individual node

### Bot Mode (Advanced - More Features)

1. Go to https://discord.com/developers/applications
2. Create a new application
3. Go to the "Bot" section and create a bot
4. Copy the bot token
5. Enable necessary bot intents (Message Content, Server Members, etc.)
6. Invite the bot to your server with required permissions
7. Get the channel ID (enable Developer Mode in Discord → right-click channel → Copy ID)
8. Configure the extension settings with bot token and default channel ID

## Nodes

### Send Discord Message
Send a message to Discord via webhook. This is the simplest method and doesn't require a bot.

**Inputs:**
- Message Content (required): The message to send
- Webhook URL: Discord webhook URL (uses extension default if blank)
- Override Username: Custom username for the webhook
- Override Avatar URL: Custom avatar for the webhook

**Outputs:**
- success: Boolean indicating if the message was sent
- method: "webhook" 
- message: Status message

### Post to Discord Channel
Post a message to a Discord channel using bot token. Requires bot setup.

**Inputs:**
- Message Content (required): The message to post
- Bot Token: Discord bot token (uses extension default if blank)
- Channel ID: Discord channel ID (uses extension default if blank)

**Outputs:**
- success: Boolean indicating if the message was sent
- method: "bot"
- message: Status message
- channelId: The channel ID posted to

### Upload Discord File
Upload a file to Discord via webhook or bot token.

**Inputs:**
- File Path (required): Path to the file to upload
- Message Content: Optional message to accompany the file
- Webhook URL: Discord webhook URL (uses extension default if blank)
- Bot Token: Discord bot token (uses extension default if blank)
- Channel ID: Discord channel ID for bot mode (uses extension default if blank)
- Override Username: Custom username for webhook mode

**Outputs:**
- success: Boolean indicating if the file was uploaded
- method: "webhook" or "bot"
- message: Status message
- fileName: Name of the uploaded file
- channelId: Channel ID (bot mode only)

## Extension Settings

Configure default values in the Extensions page:

- **Bot Token**: Default Discord bot token for bot-mode operations
- **Default Webhook URL**: Default webhook URL for message operations
- **Default Channel ID**: Default Discord channel ID for bot-mode operations

## Use Cases

- Send notifications from workflows to Discord channels
- Upload generated reports/images to Discord
- Post automated status updates
- Integrate Discord with other automation workflows

## Requirements

- discord.py==2.3.2 (included in extension)
- Discord account with appropriate permissions
- For bot mode: Discord application and bot setup

## Notes

- Webhook mode is recommended for simple use cases - no bot setup required
- Bot mode allows more advanced features but requires Discord developer setup
- File uploads support any file type allowed by Discord
- Messages support Discord's markdown formatting

## Contract (v1.0.2)

**Install:** Extensions → **Install from file** → choose `discord-integration.dsext`.

**Permissions:**
- **network**
- **filesystem**
- **secrets**

No example workflow: needs Discord bot token.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/discord-integration
python tools/deskstride_ext_cli.py pack extensions/discord-integration -o discord-integration.dsext
```
