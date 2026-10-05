# nmap-rustscan

**Version:** 1.0.2


## Contract (v1.0.2)

**Install:** Extensions → **Install from file** → choose `nmap-rustscan.dsext`.

**Permissions:**
- **network**
- **filesystem**
- **shell**

Example workflow template already ships with this extension.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/nmap-rustscan
python tools/deskstride_ext_cli.py pack extensions/nmap-rustscan -o nmap-rustscan.dsext
```

