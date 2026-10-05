# Document Converter

**Version:** 1.2.0

Universal document-to-Markdown conversion extension using anydoc (firecrawl-anydoc). Converts Word, PowerPoint, Excel, OpenDocument, RTF, EPUB, CSV, and PDF files into clean GitHub-Flavored Markdown. Includes a **Document Converter** sidebar page for one-shot converts.

## Permissions

| Permission | Why |
|---|---|
| `filesystem` | Reads the input document and writes the Markdown file you choose |
| `secrets` | Optional Firecrawl API key for OCR is stored in the OS keychain |

## Features

- **Universal Document Support** - Convert any office document format to Markdown
- **Fast Performance** - Single-digit millisecond conversion for typical files
- **Format Detection** - Automatic content-based format detection
- **GitHub-Flavored Markdown** - Clean, structured output with tables, lists, headings
- **Optional OCR** - Firecrawl Parse API integration for scanned documents
- **Offline Processing** - Core conversion works entirely offline

## Installation

1. Sidebar → **Extensions** → **Install from file...**
2. Pick `document-converter.dsext`
3. Confirm the trust warning
4. Optionally configure **Firecrawl API Key** on the extension card for OCR support
5. The **Convert Document to Markdown** node appears in the palette's "data" category
6. An **Example: Convert Document to Markdown** workflow is added on the Workflows page
7. Sidebar → **Extensions** group → **Document Converter** — pick a file, convert, preview Markdown (OCR not offered on the page)

## Example workflow

After install, open **Example: Convert Document to Markdown**. Replace the placeholder `inputFile` with a real local document path, then Run (OCR left off so it stays offline).

## Sidebar page

**Document Converter** converts a local file to Markdown and shows a short preview plus output path/stats. Leave the output blank to write next to the input with a `.md` extension.

## Supported Formats

| Format | Extensions |
|--------|------------|
| Word | `.doc`, `.docx`, `.docm` |
| PowerPoint | `.ppt`, `.pps`, `.pot`, `.pptx`, `.pptm`, `.ppsx`, `.ppsm` |
| Excel | `.xls`, `.xlsx`, `.xlsm`, `.xlsb` |
| OpenDocument | `.odt`, `.ods`, `.odp` |
| Rich Text Format | `.rtf` |
| EPUB | `.epub` |
| CSV | `.csv` |
| PDF | `.pdf` |

## Node Details

### Convert Document to Markdown
Convert any supported document format to GitHub-Flavored Markdown.

**Parameters:**
- `inputFile` - Input document file (any supported format)
- `outputPath` - Path where the Markdown file will be saved (optional, defaults to input filename with .md extension)
- `useOcr` - Enable OCR for scanned/image-only pages (requires Firecrawl API key)

**Outputs:**
- `outputPath` - Path to the generated Markdown file
- `markdown` - The full Markdown content
- `charCount` - Character count of the generated Markdown
- `lineCount` - Line count of the generated Markdown
- `format` - The input file format detected

## OCR Configuration

For OCR support on scanned documents:
1. Get a Firecrawl API key from [firecrawl.dev](https://firecrawl.dev)
2. Add the API key to the extension settings as **Firecrawl API Key**
3. Enable **Use OCR for Scanned Pages** in the node parameters

**Important:** OCR sends your document to Firecrawl's external service. This is clearly marked with a warning in the node interface.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/document-converter
python tools/deskstride_ext_cli.py scan extensions/document-converter
python tools/deskstride_ext_cli.py pack extensions/document-converter -o document-converter.dsext
```
