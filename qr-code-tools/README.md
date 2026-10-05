# QR Code Tools

Adds two node types for working with QR codes in a workflow:

- **Generate QR Code** — encode text or a URL into a QR code PNG.
- **Read QR Code** — decode an existing QR code image (PNG/JPEG/BMP/GIF)
  back to text.

## Try it

1. Sidebar → **Extensions** → **Install from file...**
2. Pick `../qr-code-tools.dsext` (the pre-built package, sibling to this
   directory) — or zip this directory yourself: `manifest.json`, `nodes/`,
   and `bridge/` go at the zip's root, no wrapping folder.
3. Confirm the trust warning. `qrcode`, `jimp`, and `jsqr` install into this
   extension's private `npm_deps/` folder — watch the streamed install log
   for real `npm` output.
4. **Generate QR Code** and **Read QR Code** appear in the palette's
   "files" category.
5. **Generate QR Code**: set **Text / URL** and **Output File**, run it,
   open the resulting PNG.
6. **Read QR Code**: set **Image File** to a QR code image (the one you
   just generated, or any other), run it, check the **text** output.

**Requires Node.js (with npm) installed and on PATH.** If it's missing,
installing this extension still succeeds up to the dependency-install
step, which then fails with a clear message in the streamed install log
(rather than a cryptic error) telling you to install Node.js.

## What each file does

- **`nodes/generate_qr_code.py`** / **`nodes/read_qr_code.py`**: the two
  node types. Each executor is a thin wrapper around one bridge call —
  build a small JSON payload, hand it to
  `node_bridge.run_extension_bridge(__file__, "bridge/<script>.cjs",
  payload)`, return the result.
- **`bridge/generate_qr.cjs`**: wraps the `qrcode` npm package —
  `QRCode.toFile(outputPath, text)`.
- **`bridge/read_qr.cjs`**: wraps `jimp` (loads/decodes the image into raw
  pixel data — pure JS, no native compilation, so it installs reliably
  everywhere Node.js does) and `jsqr` (reads the QR code out of that pixel
  data).

## How this is built (implementation note)

The actual QR encoding/decoding logic isn't reimplemented in Python — each
node calls out to a small Node.js "bridge" script that wraps the real npm
package unmodified. This is a general pattern this app supports for any
extension: see
[docs/features/extensions.md](../../../docs/features/extensions.md)'s
"Node.js bridge" section and
[docs/archive/RND_NODEJS_BRIDGE.md](../../../docs/archive/RND_NODEJS_BRIDGE.md)
for the full design reasoning, and
[docs/guide/creating-extensions.md](../../../docs/guide/creating-extensions.md)
if you want to build your own extension the same way. Practically, this is
just why Node.js needs to be installed for this specific extension to work.
