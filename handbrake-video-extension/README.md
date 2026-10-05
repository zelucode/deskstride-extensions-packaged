# HandBrake Video Transcoding

**Version:** 1.0.0

Transcode video with **HandBrakeCLI** using its presets, with live progress, and list the presets that are installed.
Requires HandBrakeCLI (the command-line build of HandBrake, https://handbrake.fr/downloads2.php). No pip packages.

## Nodes

| Node | What it does |
|---|---|
| **HandBrake Transcode** | Re-encodes a video: `HandBrakeCLI -i <in> -o <out> -f av_mp4|av_mkv|av_webm -Z "<preset>"` plus optional encoder (`-e`), quality (`-q`), your own preset file (`--preset-import-file`) and extra flags. Reports `encoding` progress as it runs. |
| **HandBrake List Presets** | Returns `presets` (`{category, name}`), `names`, `categories`, `count` so you can pick a valid, case-sensitive preset name. |

**Safe by default.** The input is never modified. The output must differ from the input, and an existing output file is not replaced unless
*Overwrite Existing Output* is on (HandBrake itself would overwrite silently). A non-zero exit shows HandBrake's last log lines;
*Timeout (minutes)* stops a stuck encode and ends the whole process tree.

## Settings

- **HandBrakeCLI Path**: file or folder. Blank = PATH, then the usual install folders.
- **Default Preset**: used when a node's Preset is blank (otherwise `Fast 1080p30`). Preset names are case-sensitive.
- **Default Output Folder**: blank = next to the input as `<name>_hb.<ext>`.

## Contract (v1.0.0)

**Install:** Extensions -> **Install from file** -> choose `handbrake-video-extension-1.0.0.dsext`.

**Permissions:**
- **filesystem**
- **shell** (runs HandBrakeCLI)

**Dependencies:** none (HandBrakeCLI must be installed separately).

**Verified with:** `tests/extensions/test_handbrake.py`, which runs the nodes against a stand-in `HandBrakeCLI` that prints HandBrake's real progress format, records its arguments, and can fail or hang on demand.
It has not been run against a real HandBrakeCLI on the build machine (not installed there); the options used are the ones in HandBrake's CLI reference.
