# realesrgan-image-upscaling

**Version:** 1.1.0


## Contract (v1.1.0)

**Install:** Extensions → **Install from file** → choose `realesrgan-image-upscaling.dsext`.

**Permissions:**
- **filesystem**
- **shell**

No example workflow: requires Real-ESRGAN / GFPGAN binaries and models.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/realesrgan-image-upscaling
python tools/deskstride_ext_cli.py pack extensions/realesrgan-image-upscaling -o realesrgan-image-upscaling.dsext
```

## Sidebar page

Sidebar **Upscaling Setup**: check whether NCNN/Python/GFPGAN paths exist before running nodes.
