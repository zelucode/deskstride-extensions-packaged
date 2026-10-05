# nosql-database-connector-extension

**Version:** 0.2.1


## Contract (v0.2.1)

**Install:** Extensions → **Install from file** → choose `nosql-database-connector-extension.dsext`.

**Permissions:**
- **network**
- **secrets**

No example workflow: needs NoSQL provider credentials.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/nosql-database-connector-extension
python tools/deskstride_ext_cli.py pack extensions/nosql-database-connector-extension -o nosql-database-connector-extension.dsext
```

## Sidebar page

Sidebar **NoSQL Browser**: test providers, list collections/keys, peek at documents.
