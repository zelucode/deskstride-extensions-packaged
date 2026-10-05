# Zod Schema Validation

**Version:** 1.2.0

A schema validation extension using the zod library via a Node.js bridge. Provides runtime type checking and detailed error reporting for JSON data validation, plus a **Schema Playground** sidebar page.

## Permissions

| Permission | Why |
|---|---|
| `shell` | Runs the zod bridge script as a child process |

## Features

- **Schema Validation**: Validate JSON data against complex schema definitions
- **Type Safety**: Runtime type checking with detailed error messages
- **Rich Schema Types**: Support for strings, numbers, booleans, objects, arrays, enums, unions, and more
- **Detailed Errors**: Get specific error messages with path information for validation failures

## Installation

1. Sidebar → **Extensions** → **Install from file...** → pick `zod-schema-validation.dsext`
2. Requires Node.js/npm on PATH so the zod package can install
3. An **Example: Validate Schema** workflow is added on the Workflows page
4. Sidebar → **Extensions** group → **Schema Playground** — paste data + schema and validate without a workflow

## Example workflow

**Example: Validate Schema** checks a sample contact object against a simple schema and branches on `valid`. Safe to run as-is — no secrets or network.

## Sidebar page

**Schema Playground** uses the same schema JSON shape as the node. Empty fields fall back to a built-in sample so you can click Validate immediately.

## Schema Format

The schema is defined as JSON with the following structure:

### Basic Types

```json
{
  "type": "string"
}
```

```json
{
  "type": "number",
  "min": 0,
  "max": 100,
  "int": true
}
```

```json
{
  "type": "boolean"
}
```

### String Constraints

```json
{
  "type": "string",
  "min": 5,
  "max": 100,
  "email": true
}
```

```json
{
  "type": "string",
  "url": true
}
```

```json
{
  "type": "string",
  "uuid": true
}
```

### Number Constraints

```json
{
  "type": "number",
  "min": 0,
  "max": 100,
  "positive": true
}
```

```json
{
  "type": "number",
  "int": true,
  "negative": true
}
```

### Objects

```json
{
  "type": "object",
  "properties": {
    "name": {"type": "string", "min": 1},
    "age": {"type": "number", "int": true, "min": 0},
    "email": {"type": "string", "email": true}
  },
  "strict": true
}
```

### Arrays

```json
{
  "type": "array",
  "items": {"type": "string"},
  "min": 1,
  "max": 10
}
```

```json
{
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": {"type": "number", "int": true},
      "name": {"type": "string"}
    }
  }
}
```

### Enums

```json
{
  "type": "enum",
  "values": ["red", "green", "blue"]
}
```

### Literals

```json
{
  "type": "literal",
  "value": "admin"
}
```

### Optional and Nullable

```json
{
  "type": "optional",
  "inner": {"type": "string"}
}
```

```json
{
  "type": "nullable",
  "inner": {"type": "number"}
}
```

### Unions

```json
{
  "type": "union",
  "options": [
    {"type": "string"},
    {"type": "number"}
  ]
}
```

### Any/Unknown

```json
{
  "type": "any"
}
```

```json
{
  "type": "unknown"
}
```

## Usage Examples

### Example 1: Simple User Validation

**Schema:**
```json
{
  "type": "object",
  "properties": {
    "name": {"type": "string", "min": 1},
    "age": {"type": "number", "int": true, "min": 0},
    "email": {"type": "string", "email": true}
  }
}
```

**Valid Data:**
```json
{
  "name": "John Doe",
  "age": 30,
  "email": "john@example.com"
}
```

**Invalid Data:**
```json
{
  "name": "",
  "age": -5,
  "email": "not-an-email"
}
```

### Example 2: Product Validation

**Schema:**
```json
{
  "type": "object",
  "properties": {
    "id": {"type": "number", "int": true, "positive": true},
    "name": {"type": "string", "min": 1, "max": 100},
    "price": {"type": "number", "positive": true},
    "category": {
      "type": "enum",
      "values": ["electronics", "clothing", "food", "other"]
    },
    "tags": {
      "type": "array",
      "items": {"type": "string"},
      "min": 0,
      "max": 10
    }
  }
}
```

### Example 3: API Response Validation

**Schema:**
```json
{
  "type": "object",
  "properties": {
    "success": {"type": "boolean"},
    "data": {
      "type": "union",
      "options": [
        {"type": "object"},
        {"type": "array"}
      ]
    },
    "error": {
      "type": "optional",
      "inner": {"type": "string"}
    }
  }
}
```

## Output

The node returns the following outputs:

- **valid**: Boolean indicating whether the data passed validation
- **errors**: Array of error objects (if validation failed), each containing:
  - **path**: Dot-separated path to the invalid field
  - **message**: Human-readable error message
  - **code**: zod error code
- **validatedData**: The validated and potentially transformed data (if validation passed)

## Error Handling

If the data fails validation, the node will still succeed but return `valid: false` with detailed error information. You can use conditional branching in your workflow to handle validation failures.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/zod-schema-validation
python tools/deskstride_ext_cli.py scan extensions/zod-schema-validation
python tools/deskstride_ext_cli.py pack extensions/zod-schema-validation -o zod-schema-validation.dsext
```

## License

This extension uses the MIT-licensed zod npm package. See https://github.com/colinhacks/zod for the zod license and documentation.
