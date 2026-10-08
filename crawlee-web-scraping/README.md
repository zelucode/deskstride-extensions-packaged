# Crawlee Web Scraping

**Version:** 1.0.2

A web scraping and crawling extension using the Crawlee library via a Node.js bridge. Provides HTTP crawling, data extraction with CSS selectors, and multi-page link following.

## Permissions

| Permission | Why |
|---|---|
| `network` | Fetches pages from the URLs you give it |
| `shell` | Runs the Crawlee bridge script as a child process |

## Features

- **HTTP Crawling**: Fast web scraping using CheerioCrawler
- **Data Extraction**: Extract text, links, images, and custom fields with CSS selectors
- **Website Crawling**: Crawl multiple pages by following links
- **URL Pattern Matching**: Filter which URLs to crawl using regex patterns
- **Concurrency Control**: Configurable crawl limits

## Installation

1. Sidebar → **Extensions** → **Install from file...** → pick `crawlee-web-scraping.dsext`
2. Requires Node.js/npm on PATH so the declared npm packages can install into this extension's private folder

No example workflow ships with this extension — a realistic first-run demo needs a live site crawl plus the Node/Crawlee install, which is too heavy for a safe self-contained starter.

## Usage

### Scrape URL Node

Scrapes a single URL and extracts data based on CSS selectors.

**Parameters:**
- **url**: The URL to scrape (required)
- **selector**: JSON object with CSS selectors (optional)
- **maxPages**: Maximum number of pages to scrape (default: 10)
- **followLinks**: Whether to follow links on the page (default: false)

**Selector Format:**
```json
{
  "title": "h1",
  "text": "p",
  "links": "a",
  "images": "img",
  "custom": [
    {"selector": ".author", "name": "author"},
    {"selector": ".date", "name": "publishDate"}
  ]
}
```

**Example Usage:**
1. **Basic Page Scraping:**
   - URL: `https://example.com`
   - Selector: `{"title": "h1", "text": "p"}`
   - This will extract the page title and all paragraph text

2. **Link Extraction:**
   - URL: `https://example.com`
   - Selector: `{"links": "a"}`
   - This will extract all links with their text and URLs

3. **Image Extraction:**
   - URL: `https://example.com`
   - Selector: `{"images": "img"}`
   - This will extract all image URLs

4. **Custom Data Extraction:**
   - URL: `https://example.com`
   - Selector: `{"custom": [{"selector": ".product-name", "name": "name"}, {"selector": ".price", "name": "price"}]}`
   - This will extract custom fields using CSS selectors

### Crawl Website Node

Crawls a website starting from a URL, following links and extracting data from multiple pages.

**Parameters:**
- **startUrl**: The URL to start crawling from (required)
- **maxPages**: Maximum number of pages to crawl (default: 20)
- **urlPattern**: Regex pattern to filter which URLs to crawl (optional)
- **selector**: JSON object with CSS selectors (optional)

**Example Usage:**
1. **Basic Website Crawling:**
   - Start URL: `https://example.com`
   - Max Pages: 20
   - This will crawl up to 20 pages starting from the given URL

2. **Pattern-Based Crawling:**
   - Start URL: `https://example.com`
   - URL Pattern: `https://example.com/blog/.*`
   - Max Pages: 50
   - This will only crawl URLs matching the pattern

3. **Data Extraction During Crawling:**
   - Start URL: `https://example.com`
   - Selector: `{"title": "h1", "text": "article p"}`
   - Max Pages: 30
   - This will extract titles and article text from each crawled page

## Output

Both nodes return the following outputs:

- **results**: Array of scraped data objects, each containing:
  - **url**: The URL that was scraped
  - **data**: The extracted data based on selectors
- **totalPages**: Total number of pages scraped
- **firstResult**: The first result object (for scrape_url node)
- **urls**: Array of all URLs scraped (for crawl_website node)

## Advanced Features

### Selector Configuration

The selector configuration supports several built-in field types:

- **title**: CSS selector for page title
- **text**: CSS selector for main text content
- **links**: CSS selector for links (extracts both text and URL)
- **images**: CSS selector for images (extracts URLs)
- **custom**: Array of custom selector objects with `selector` and `name` fields

### URL Pattern Matching

Use regex patterns to control which URLs are crawled:

```regex
https://example.com/blog/.*     # Match all blog posts
https://example.com/products/.*  # Match all product pages
.*\.html$                       # Match all HTML files
```

## Error Handling

- If the URL is invalid or unreachable, the node will fail with an error
- If CSS selectors don't match any elements, the corresponding fields will be empty
- Network timeouts and errors are handled gracefully with descriptive error messages

## Performance Considerations

- **Concurrency**: The crawler uses single-threaded execution by default for stability
- **Memory Usage**: Large crawls may consume significant memory; consider reducing maxPages for memory-constrained environments
- **Rate Limiting**: Be respectful to target servers by using appropriate maxPages limits and delays

## Use Cases

1. **Content Aggregation**: Scrape news sites, blogs, or content platforms
2. **Price Monitoring**: Monitor product prices across e-commerce sites
3. **Data Extraction**: Extract structured data from unstructured web pages
4. **SEO Analysis**: Analyze website structure and content
5. **Research**: Collect data for research and analysis purposes

## Limitations

- Some sites may block automated scraping; respect robots.txt and terms of service
- Dynamic content that only appears after client-side scripting may need a browser-based crawler (not wired in this package)

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/crawlee-web-scraping
python tools/deskstride_ext_cli.py scan extensions/crawlee-web-scraping
python tools/deskstride_ext_cli.py pack extensions/crawlee-web-scraping -o crawlee-web-scraping.dsext
```

## License

This extension uses the Apache-2.0 licensed Crawlee npm package. See https://github.com/apify/crawlee for the Crawlee license and documentation.
