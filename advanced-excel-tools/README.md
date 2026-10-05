# Advanced Excel Tools

Adds Excel nodes for formulas, conditional formatting, charts, pivot tables,
cell styling, column auto-fit, sheet protection, and XLS-to-XLSX conversion,
plus an Excel Inspector sidebar page.

**Version:** 1.1.0

## Install

1. Open **Extensions** → **Install from file**
2. Choose `advanced-excel-tools.dsext`
3. Confirm the install prompt (dependencies install into this extension's private folder)
4. Optional: set **Default Output Folder** on the extension card

## Permissions

- **filesystem** — nodes read and write workbooks you select (and the inspector opens a path you enter)

## Example workflow

On install, **Example: Chart from Excel** is added to Workflows. Point the
placeholder paths at an `.xlsx` with labels in column A and numbers in column B,
then Run to apply a formula and embed a bar chart.

## Nodes

| Node | What it does |
|------|----------------|
| Apply Formula to Range | Writes a formula into a cell or range |
| Conditional Format | Color-scales or highlights a range |
| Generate Chart | Embeds a bar, line, or pie chart |
| Pivot Table | Builds a summary sheet from row data |
| Set Cell Style | Font, fill, borders, number formats |
| Autofit Columns | Resizes columns to content |
| Protect Sheet | Locks a sheet with an optional password |
| Convert XLS to XLSX | Upgrades legacy `.xls` workbooks |

## Develop / pack

```bash
python tools/deskstride_ext_cli.py lint extensions/advanced-excel-tools
python tools/deskstride_ext_cli.py scan extensions/advanced-excel-tools
python tools/deskstride_ext_cli.py check extensions/advanced-excel-tools
python tools/deskstride_ext_cli.py pack extensions/advanced-excel-tools -o advanced-excel-tools.dsext
```
