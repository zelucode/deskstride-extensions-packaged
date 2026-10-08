# RSS Feed Reader

**Version:** 1.1.1

Polls RSS/Atom feeds and returns new items since the last run.

## Permissions

| Permission | Why |
|---|---|
| `network` | Fetches the feed URL you configure |

## Features

- **RSS/Atom Support**: Works with both RSS and Atom feed formats
- **State Tracking**: Remembers seen item IDs per node instance so re-runs only emit new items
- **Configurable Limits**: Control maximum items returned per run
- **Loop-Ready**: Returns new items as an array for per-item processing

## Installation

1. Sidebar → **Extensions** → **Install from file...** → pick `rss-feed-reader.dsext`
2. The **Read RSS Feed** node appears in the palette's "web" category
3. An **Example: Read RSS Feed** workflow is added on the Workflows page

## Example workflow

**Example: Read RSS Feed** polls NASA's public breaking-news feed (5 items max) and branches on whether any new items appeared. Safe defaults — no secrets.

## Node: Read RSS Feed

**Type**: `read_rss_feed`  
**Category**: Web

### Parameters

- **Feed URL** (required): The RSS or Atom feed URL to poll
- **Max Items to Return** (optional): Maximum number of items to return per run (default: 10)

### Outputs

- **newItems**: Array of new feed items since last run
- **totalItems**: Total number of items in the feed
- **newItemCount**: Number of new items found
- **feedTitle**: Title of the feed
- **feedUpdated**: Last updated timestamp of the feed

### New Item Structure

Each item in `newItems` contains:
- `id`: Unique identifier (GUID or link)
- `title`: Item title
- `link`: Item URL
- `published`: Publication timestamp
- `summary`: Item summary/description
- `author`: Item author
- `tags`: Array of topic tags

## Usage Example

1. Install the extension
2. Add a "Read RSS Feed" node to your workflow
3. Configure the Feed URL (e.g., `https://example.com/feed.xml`)
4. Set Max Items to your desired limit
5. Connect the output to a Loop node to process each new item
6. Use a Condition node to check if `newItemCount > 0` before processing

## State Management

The node tracks seen item IDs per node instance across workflow runs:
- Different node instances do not share state
- State survives workflow restarts but clears if the node is deleted/recreated
- Stores up to 1000 most recent item IDs to prevent unbounded growth

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/rss-feed-reader
python tools/deskstride_ext_cli.py scan extensions/rss-feed-reader
python tools/deskstride_ext_cli.py pack extensions/rss-feed-reader -o rss-feed-reader.dsext
```
