# text-analysis-nlp

**Version:** 1.2.0


## Contract (v1.2.0)

**Install:** Extensions → **Install from file** → choose `text-analysis-nlp.dsext`.

**Permissions:**
- **network**
- **shell**

See templates/ for the example workflow added on install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/text-analysis-nlp
python tools/deskstride_ext_cli.py pack extensions/text-analysis-nlp -o text-analysis-nlp.dsext
```

## Sidebar page

Sidebar **NLP Models**: TextBlob/spaCy status and one-click spaCy model download.
