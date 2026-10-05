# QR Code & Barcode Tools Extension

**Version:** 1.1.0

A QR code and barcode generation and scanning extension for DeskStride. Supports generating QR codes, various barcode formats, and scanning codes from images.

## Features

- **Generate QR Code**: Create QR codes from text/URL with custom sizes, colors, and error correction
- **Generate Barcode**: Create barcodes in Code128, Code39, EAN13, EAN8, and UPC formats
- **Scan QR/Barcode**: Scan and decode QR codes and barcodes from image files

## Setup

No external setup required. The extension includes all necessary Python libraries:
- qrcode==7.4.2 (QR code generation)
- python-barcode==0.15.1 (barcode generation)
- pyzbar==0.1.9 (code scanning)
- Pillow==10.0.0 (image processing)

### Extension Settings

Configure default values in the Extensions page:

- **Default QR Code Size**: Default size for generated QR codes in pixels
- **Default Barcode Format**: Default format for barcode generation (code128, code39, ean13, ean8, upc)

## Nodes

### Generate QR Code
Generate a QR code from text or URL data with customizable appearance.

**Inputs:**
- Text/URL (required): The text or URL to encode in the QR code
- Output Path (required): Path where the QR code image will be saved
- Size (pixels): Size of the QR code in pixels (default: 300)
- Error Correction: Error correction level (L=Low, M=Medium, Q=Quartile, H=High)
- Fill Color: Color of the QR code (default: black)
- Background Color: Background color (default: white)

**Outputs:**
- success: Boolean indicating if QR code was generated
- outputPath: Path to the generated QR code image
- text: The text encoded in the QR code
- size: The size used
- message: Status message

### Generate Barcode
Generate a barcode from text or numeric data in various formats.

**Inputs:**
- Text/Number (required): The text or number to encode in the barcode
- Output Path (required): Path where the barcode image will be saved
- Barcode Format: Format to use (code128, code39, ean13, ean8, upc)
- Module Width: Width of each bar module in pixels (default: 2)
- Module Height: Height of the barcode in pixels (default: 100)

**Outputs:**
- success: Boolean indicating if barcode was generated
- outputPath: Path to the generated barcode image
- text: The text encoded in the barcode
- format: The barcode format used
- message: Status message

### Scan QR/Barcode
Scan and decode QR codes and barcodes from an image file.

**Inputs:**
- Image Path (required): Path to the image file containing QR codes or barcodes

**Outputs:**
- success: Boolean indicating if scan completed
- imagePath: Path to the scanned image
- codeCount: Number of codes found
- codes: Array of decoded code objects with type, data, and position info
- message: Status message

**Code Object Structure:**
```json
{
  "type": "QRCODE",
  "data": "https://example.com",
  "rect": {
    "left": 100,
    "top": 50,
    "width": 200,
    "height": 200
  }
}
```

## Supported Formats

### QR Codes
- Standard QR codes with error correction levels L, M, Q, H
- Custom sizes and colors
- Automatic error correction for damaged codes

### Barcodes
- **Code128**: Most common 1D barcode, supports alphanumeric
- **Code39**: Older standard, supports alphanumeric
- **EAN13**: Standard retail product barcode (13 digits)
- **EAN8**: Shorter retail product barcode (8 digits)
- **UPC**: North American retail barcode (12 digits)

## Use Cases

- **Inventory Management**: Generate barcodes for products, scan for tracking
- **Contact Sharing**: Generate QR codes with contact information (vCard format)
- **URL Shortening**: Create QR codes that link to websites
- **Document Indexing**: Add barcodes to documents for easy scanning and retrieval
- **Event Management**: Generate QR codes for tickets, check-in systems
- **Asset Tracking**: Label equipment with barcodes/QR codes for tracking
- **Marketing Materials**: Create QR codes for print materials linking to digital content

## Error Correction Levels for QR Codes

- **L (Low)**: ~7% error correction, good for clean environments
- **M (Medium)**: ~15% error correction, recommended for general use
- **Q (Quartile)**: ~25% error correction, for damaged codes
- **H (High)**: ~30% error correction, maximum redundancy

## Tips

- Use higher error correction (Q or H) for QR codes that might get damaged or printed
- Code128 is the most versatile barcode format for general use
- EAN13/UPC for retail products (require specific digit counts)
- Scan multiple codes in a single image - the node returns all found codes
- QR codes work best with high contrast (black on white or white on black)

## Requirements

- All Python dependencies are included in the extension
- No external software required
- Works on all platforms (Windows, macOS, Linux)

## Notes

- QR codes can encode URLs, text, contact info, WiFi credentials, and more
- Barcode formats have specific character requirements (e.g., EAN13 requires exactly 13 digits)
- Scanning works with multiple codes in a single image
- Generated images are in PNG format
- For best scanning results, use high-resolution images with good lighting

## Contract (v1.1.0)

**Install:** Extensions → **Install from file** → choose `qr-barcode-tools.dsext`.

**Permissions:**
- **filesystem**

See templates/ for the example workflow added on install.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/qr-barcode-tools
python tools/deskstride_ext_cli.py pack extensions/qr-barcode-tools -o qr-barcode-tools.dsext
```
