# Image Processing

**Version:** 1.0.0

Resize, crop, convert, watermark and thumbnail images with [Pillow](https://python-pillow.org), one image at a time or a whole
folder at once. **Originals are never modified or overwritten**: every node writes a new file.

## Nodes

All single-image nodes take **Input Image** and an optional **Output Path**. Leave the output blank and the result is saved next to
the input (or in the extension's *Default Output Folder*) with a suffix such as `_resized`. EXIF orientation is applied, so phone photos
come out the right way up.

| Node | What it does | Main fields |
|---|---|---|
| **Resize Image** | Change size | Width, Height, Resize Mode (`fit` keeps the aspect ratio inside the box, `fill` covers the box and crops the overflow, `stretch` forces the exact size). Set only one of width/height and the other follows. |
| **Crop Image** | Cut a rectangle | Crop From (`center` or `custom`), X, Y, Crop Width, Crop Height. A box outside the image is an error, not a silent clamp. |
| **Convert Image Format** | Change format | Target Format (png, jpeg, webp, bmp, tiff, gif), Quality, Background Colour (replaces transparency when the format cannot store it, e.g. JPEG). |
| **Add Watermark** | Stamp text and/or a logo | Text, Watermark Image, Position (9 spots), Opacity, Margin, Text Size, Text Colour, Logo Width (% of image). |
| **Create Thumbnail** | Shrink to a max size | Max Size (longest side). Never enlarges. |
| **Batch Process Images** | Run one of the above on every image in a folder | Input Folder, Operation, Output Folder, Include Subfolders, plus the fields of the chosen operation. |

The batch node writes into `Output Folder` (default: a `processed` folder inside the input folder), keeps sub-folder structure when
*Include Subfolders* is on, never reprocesses its own output, and never lets two files overwrite each other. A file that fails is listed
under `failed` with the reason and the rest carry on.

## Outputs

Single nodes: `outputPath`, `width`, `height`, `format`, `fileSize`, `originalWidth`, `originalHeight`, `inputPath`.
Batch: `outputFolder`, `operation`, `total`, `processedCount`, `failedCount`, `processed` (list of paths), `failed` (list of `{file, error}`).

## Settings

- **Default Output Folder** (`defaultOutputDir`): where results go when a node's output path is blank.
- **Default JPEG/WebP Quality** (`defaultQuality`): 1-100, used when a node's Quality is blank (90 if unset).

## Contract (v1.0.0)

**Install:** Extensions → **Install from file** → choose `image-processing-extension-1.0.0.dsext`.

**Permissions:**
- **filesystem**

**Dependencies:** `Pillow==10.4.0` (installed into the extension's private `deps/` folder).

**Supported formats:** read and write PNG, JPEG, WebP, BMP, TIFF, GIF (first frame of animated GIFs only).

No example workflow is bundled.

**Develop:** node logic is covered by `tests/extensions/test_image_processing.py` in the source repo.
