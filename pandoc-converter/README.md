# pandoc-converter

**Version:** 1.0.1


## Contract (v1.0.1)

**Install:** Extensions → **Install from file** → choose `pandoc-converter.dsext`.

**Permissions:**
- **filesystem**
- **shell**

No example workflow: requires Pandoc binary on PATH.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/pandoc-converter
python tools/deskstride_ext_cli.py pack extensions/pandoc-converter -o pandoc-converter.dsext
```

