# database-connector-extension

**Version:** 0.2.1


## Contract (v0.2.1)

**Install:** Extensions → **Install from file** → choose `database-connector-extension.dsext`.

**Permissions:**
- **network**
- **filesystem**
- **secrets**

No example workflow: needs live DB credentials; SQLite path alone is not a full demo story.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/database-connector-extension
python tools/deskstride_ext_cli.py pack extensions/database-connector-extension -o database-connector-extension.dsext
```

## Sidebar page

Sidebar **Database Browser**: test connections, list tables, inspect schemas, preview SELECT queries.
