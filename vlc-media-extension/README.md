# VLC Media Convert & Control

**Version:** 1.0.0

Convert media with VLC, play files in VLC, and control playback (pause, seek, volume...) from a workflow. Requires **VLC** (https://www.videolan.org/vlc/).
No pip packages: conversion runs `vlc` itself and control talks to VLC's built-in web interface.

## Nodes

| Node | What it does |
|---|---|
| **VLC Convert Media** | Presets: `mp4`, `mkv` (H.264 + AAC), `webm` (VP8 + Vorbis), `mp3`, `ogg`, `flac`, `wav` (audio only: video is dropped), or `custom` with your own `--sout` chain containing `{output}`. Video and audio bitrate, overwrite guard, timeout. |
| **VLC Play** | Opens a file or URL in VLC and leaves it running. With *Enable Remote Control* (default) it starts VLC's web interface on **127.0.0.1 only**, protected by the extension's password. Options: start time, fullscreen, loop, close when finished, extra VLC arguments. |
| **VLC Control** | Drives that VLC: `status`, `pause`, `resume`, `toggle`, `stop`, `next`, `previous`, `seek` (`90`, `+10`, `-10`, `50%`, `1:02:30`), `volume` (0-200, 100 = normal), `fullscreen`. Returns `state`, `positionSeconds`, `lengthSeconds`, `volumePercent`, `filename`. |

Typical use: **VLC Play** (remote on) -> **VLC Control** `seek` / `pause` / `volume` later in the workflow.

## Settings

- **VLC Path**: blank = PATH, then the usual install folders.
- **Remote Control Password** (secret): required for VLC Play with remote control, and for VLC Control. VLC's web interface refuses to run without one; it also stops other programs on this PC from steering VLC.
- **Remote Control Port**: default 8089. VLC Play refuses a port that is already in use instead of fighting for it.
- **Default Output Folder** for conversions.

## Limits worth knowing

- **Very small videos can lose their video track.** In testing, VLC's transcoder intermittently produced an audio-only file for inputs of about 160x120; 320x240 and larger were reliable. VLC still reports success, so check the result if you convert tiny clips (the **ffmpeg-media-bridge** extension is a better fit for those).
- Conversion has no percentage: progress is reported at start and finish only.
- Replacing an existing output requires *Overwrite Existing Output*; the input file is never overwritten.

## Contract (v1.0.0)

**Install:** Extensions -> **Install from file** -> choose `vlc-media-extension-1.0.0.dsext`.

**Permissions:**
- **filesystem**
- **shell** (runs VLC)
- **network** (localhost only, to VLC's web interface)

**Dependencies:** none (VLC must be installed separately).

**Verified with:** `tests/extensions/test_vlc.py`, including end-to-end runs against real **VLC 3.0.21**: all seven conversion presets (checked with ffprobe), and Play -> status / pause / seek / resume / stop with a wrong-password and port-in-use check.
