# CSV JSON Data Tools

**Version:** 1.1.0

Four data transformation nodes for CSV and JSON arrays of objects:

- **Convert CSV JSON Dataset** reads a file or inline data and writes CSV or JSON.
- **Map and Type-Convert Records** renames fields and converts selected values.
- **Deduplicate Records** keeps the first record for each selected key combination.
- **Profile and Validate Dataset** reports missing values, unique counts, observed types, and optional JSON Schema errors.

## Input shape

JSON input must be an array of objects. An object containing a `records` or
`data` array is also accepted. CSV input must contain a header row.

Nodes accept either an input file or inline text. When `Input Format` is
`auto`, file extensions or the inline payload shape are used for detection.

## JSON Schema validation

`Profile and Validate Dataset` accepts a JSON Schema in `schemaJson`. The
schema is applied to the complete records array, so use an array schema when
validating row shape, for example:

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "required": ["email"],
    "properties": {
      "email": {"type": "string", "format": "email"}
    }
  }
}
```

The node returns validation results in its output rather than failing for
ordinary schema mismatches. Invalid JSON in the schema itself fails the node.

## Limitations

- CSV values remain strings until passed through the mapping node.
- Mapping applies one target type to all mapped fields in a node instance.
- Deduplication keeps the first occurrence and compares key values exactly.
- Install-test the packaged extension in DeskStride to verify runtime
  dependency installation and palette registration.

## Contract (v1.1.0)

**Install:** Extensions → **Install from file** → choose `csv-json-data-tools.dsext`.

**Permissions:**
- **filesystem**

See templates/ for the example workflow added on install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/csv-json-data-tools
python tools/deskstride_ext_cli.py pack extensions/csv-json-data-tools -o csv-json-data-tools.dsext
```
