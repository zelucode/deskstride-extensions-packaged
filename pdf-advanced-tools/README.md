# PDF Advanced Tools

**Version:** 2.2.0

Production-ready PDF manipulation extension — form filling, page merge/split, and form flattening, plus a **PDF Advanced Inspector** sidebar page.

## Permissions

| Permission | Why |
|---|---|
| `filesystem` | Reads and writes the PDF paths you select |

## Features

- **Fill PDF Form** - Fill form fields in PDF documents with provided values
- **Flatten PDF Form** - Flatten form fields and annotations into permanent page content
- **Merge PDF Pages** - Merge pages from multiple PDF files into a single document
- **Split PDF Pages** - Split a PDF document into individual page files

## Installation

1. Sidebar → **Extensions** → **Install from file...**
2. Pick `pdf-advanced-tools.dsext`
3. Confirm the trust warning
4. Optionally set **Default Output Folder** on the extension card
5. The new PDF nodes appear in the palette's "pdf" category
6. An **Example: Merge PDF Pages** workflow is added on the Workflows page
7. Sidebar → **Extensions** group → **PDF Advanced Inspector** — inspect pages/form fields, fill, or flatten without a workflow

## Example workflow

After install, open **Example: Merge PDF Pages**. Put two real PDF paths in the variable (one per line), set an output path, then Run.

## Sidebar page

**PDF Advanced Inspector** lets you pick a PDF path to see page count and form field names, fill fields from JSON into a new file, or flatten form widgets. Fill/Flatten never overwrite the input.

## Node Details

### Fill PDF Form
Fill form fields in a PDF document with provided values.

**Parameters:**
- `inputPdf` - PDF file containing form fields to fill
- `formData` - JSON object with field names as keys and values to fill
- `outputPath` - Path where the filled PDF will be saved

**Example formData:**
```json
{
  "name": "John Doe",
  "email": "john@example.com",
  "address": "123 Main St"
}
```

### Flatten PDF Form
Flatten form fields and annotations into permanent page content, making them no longer editable as form fields.

**Parameters:**
- `inputPdf` - PDF file with form fields to flatten
- `outputPath` - Path where the flattened PDF will be saved

### Merge PDF Pages
Merge pages from multiple PDF files into a single document.

**Parameters:**
- `inputPdfs` - PDF files to merge (one path per line, in order)
- `outputPath` - Path where the merged PDF will be saved

### Split PDF Pages
Split a PDF document into individual page files.

**Parameters:**
- `inputPdf` - PDF file to split into individual pages
- `outputDir` - Folder where individual page files will be saved
- `baseName` - Base name for output files (e.g., 'page' creates 'page_1.pdf', 'page_2.pdf', etc.)

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/pdf-advanced-tools
python tools/deskstride_ext_cli.py scan extensions/pdf-advanced-tools
python tools/deskstride_ext_cli.py pack extensions/pdf-advanced-tools -o pdf-advanced-tools.dsext
```
