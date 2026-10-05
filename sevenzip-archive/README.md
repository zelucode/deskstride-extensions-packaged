# 7-Zip Archive Management Extension

**Version:** 1.1.1

A 7-Zip integration for DeskStride that enables creating, extracting, and managing archives with support for password protection and batch operations.

## Features

- **Create 7-Zip Archive**: Create archives in multiple formats (7z, zip, gzip, bzip2, tar) with compression control
- **Extract 7-Zip Archive**: Extract archives with password support and selective file extraction
- **List 7-Zip Archive**: View archive contents with detailed file information

## Setup

### Install 7-Zip

1. Download and install 7-Zip from https://www.7-zip.org/
2. Note the installation path (default: `C:\Program Files\7-Zip\7z.exe` on Windows)
3. Configure the extension settings with the 7-Zip executable path if not in system PATH

### Extension Settings

Configure in the Extensions page:

- **7-Zip Executable Path**: Path to 7z.exe (e.g., `C:\Program Files\7-Zip\7z.exe`). Leave blank if 7z is in system PATH.
- **Default Archive Password**: Default password for encrypted archives (can be overridden per node)
- **Default Compression Level**: Default compression level (0=Store, 1=Fastest, 3=Fast, 5=Normal, 7=Maximum, 9=Ultra)

## Nodes

### Create 7-Zip Archive
Create an archive using 7-Zip CLI with support for multiple formats and password protection.

**Inputs:**
- Files/Folders to Archive (required): One file/folder path per line. Supports wildcards like `*.txt`
- Output Archive Path (required): Path where the archive will be created
- Archive Format: Format to use (7z, zip, gzip, bzip2, tar)
- Archive Password: Password for encrypted archive (uses extension default if blank)
- Compression Level: 0=Store, 1=Fastest, 3=Fast, 5=Normal, 7=Maximum, 9=Ultra
- Recursive: Include subdirectories recursively

**Outputs:**
- success: Boolean indicating if archive was created
- archivePath: Path to the created archive
- fileCount: Number of items archived
- format: Archive format used
- message: Status message

### Extract 7-Zip Archive
Extract an archive using 7-Zip CLI with password support and selective extraction.

**Inputs:**
- Archive Path (required): Path to the archive file to extract
- Output Directory: Directory where files will be extracted (blank = archive's directory)
- Archive Password: Password for encrypted archive (uses extension default if blank)
- Extract Specific Files: Specific files/folders to extract (space-separated, blank = all)
- Overwrite Existing Files: Overwrite files that already exist in output directory

**Outputs:**
- success: Boolean indicating if extraction succeeded
- archivePath: Path to the extracted archive
- outputDir: Directory where files were extracted
- extractedCount: Number of files extracted
- message: Status message

### List 7-Zip Archive
List the contents of an archive with detailed file information.

**Inputs:**
- Archive Path (required): Path to the archive file to list
- Archive Password: Password for encrypted archive (uses extension default if blank)
- Output Format: Format for textOutput (text or JSON)

**Outputs:**
- success: Boolean indicating if listing succeeded
- archivePath: Path to the listed archive
- fileCount: Number of files in archive
- files: Array of file information objects
- textOutput: Formatted text or JSON output

## Use Cases

- Automated backup workflows with compression
- Batch file compression and extraction
- Password-protected archive creation for sensitive data
- Log file archival and rotation
- Software distribution packaging
- Selective file extraction from large archives

## Supported Formats

- **7z**: Native 7-Zip format (best compression)
- **zip**: Universal ZIP format (maximum compatibility)
- **gzip**: GZIP compression (common on Unix/Linux)
- **bzip2**: BZIP2 compression (better compression than gzip)
- **tar**: TAR archiving (no compression, often used with gzip)

## Password Security

Passwords are handled securely:
- Extension settings can mark passwords as secret (encrypted storage)
- Node-level passwords override extension defaults
- Passwords are never logged or exposed in error messages
- Use strong passwords for sensitive archives

## Compression Levels

- **0**: Store (no compression)
- **1**: Fastest (lowest compression, fastest speed)
- **3**: Fast (good balance)
- **5**: Normal (default, recommended)
- **7**: Maximum (high compression, slower)
- **9**: Ultra (highest compression, slowest)

## Requirements

- 7-Zip installed (https://www.7-zip.org/)
- No Python dependencies (uses 7-Zip CLI directly)

## Notes

- 7-Zip must be installed and accessible via system PATH or specified in extension settings
- Wildcards are supported in file specifications (e.g., `*.txt`, `logs/*.log`)
- Large archives may take significant time depending on compression level
- Extraction preserves file permissions and timestamps
- Password-protected archives require the same password for extraction

## Contract (v1.1.1)

**Install:** Extensions → **Install from file** → choose `sevenzip-archive.dsext`.

**Permissions:**
- **filesystem**
- **shell**
- **secrets**

No example workflow: requires 7-Zip installed on PATH.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/sevenzip-archive
python tools/deskstride_ext_cli.py pack extensions/sevenzip-archive -o sevenzip-archive.dsext
```
