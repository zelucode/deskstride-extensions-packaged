# FFmpeg Advanced Media Processing Bridge Extension

**Version:** 1.1.0

A comprehensive media processing bridge for DeskStride powered by FFmpeg and FFprobe. Handles advanced audio and video tasks including EBU R128 loudness normalization, high-quality GIF generation with custom 2-pass color palettes, lossless audio extraction, video/audio clipping, format probing, video thumbnail generation, and complex filter graphs.

## Features

- **Normalize Audio (EBU R128)**: Standardized loudness normalization using the `loudnorm` filter (target integrated loudness `I`, loudness range `LRA`, and true peak `TP`) for podcasts, broadcasting, and streaming. Preserves video streams untouched.
- **Extract Audio Stream**: Pull audio tracks from video or multi-track containers into MP3, AAC, WAV, FLAC, M4A, OGG, or Opus with custom bitrates and sample rates.
- **Create GIF**: Generate crisp, high-fidelity animated GIFs with 2-pass palette generation (`palettegen` / `paletteuse`) to eliminate color banding and dithering artifacts.
- **Cut Media Clip**: Instant lossless keyframe trimming (`-c copy`) or frame-accurate re-encoding cuts with start time, end time, and duration parameters.
- **Probe Media Info**: Inspect containers, video/audio streams, codecs, dimensions, framerates, bitrates, duration, and metadata using `ffprobe`.
- **Generate Thumbnail**: Capture high-quality video snapshot frames at any timestamp, with optional resolution resizing.
- **Run Custom Filter**: Run arbitrary video filters (`-vf`), audio filters (`-af`), or multi-input complex filter graphs (`-filter_complex`).

## Prerequisites

FFmpeg and FFprobe are universally available on systems running media downloaders (such as `yt-dlp`).
If FFmpeg is not already installed:
1. Download from [ffmpeg.org/download.html](https://ffmpeg.org/download.html) (or install via `winget install Gyan.FFmpeg`, `brew install ffmpeg`, or `apt install ffmpeg`).
2. Verify with `ffmpeg -version` and `ffprobe -version`.

## Extension Settings

Configure via the DeskStride Extensions page:

- **FFmpeg Executable Path (`ffmpegPath`)**: Custom path to `ffmpeg` or `ffmpeg.exe`. Leave blank to use system `PATH`.
- **FFprobe Executable Path (`ffprobePath`)**: Custom path to `ffprobe` or `ffprobe.exe`. Leave blank to use system `PATH`.
- **Default Output Folder (`defaultOutputDir`)**: Directory where processed files will be saved when a node's output path is omitted.

---

## Nodes

### 1. FFmpeg Normalize Audio (`ffmpeg_normalize_audio`)
Normalizes audio loudness according to broadcast/streaming standards (EBU R128 / loudnorm).

**Inputs:**
- `inputPath` (file, required): Source audio or video file.
- `outputPath` (text): Output destination path. Defaults to `<filename>_normalized.<ext>`.
- `targetI` (text, default `-23.0`): Target integrated loudness in LUFS (`-23.0` for EBU R128, `-16.0` for podcasts/web).
- `targetLRA` (text, default `7.0`): Target loudness range in LU.
- `targetTP` (text, default `-2.0`): Maximum true peak in dBTP.
- `audioCodec` (select): Audio encoder (`auto`, `aac`, `mp3`, `flac`, `pcm_s16le`).
- `audioBitrate` (text, default `192k`): Bitrate for lossy codecs.

**Outputs:**
- `outputPath`: Path to the normalized media file.
- `targetI`, `targetLRA`, `targetTP`: Configured normalization parameters.
- `fileSize`: Output file size in bytes.

---

### 2. FFmpeg Extract Audio (`ffmpeg_extract_audio`)
Extracts audio from video or audio files into the target format.

**Inputs:**
- `inputPath` (file, required): Source media file.
- `outputPath` (text): Output audio path. Defaults to `<filename>_audio.<format>`.
- `format` (select, default `mp3`): Target format (`mp3`, `aac`, `wav`, `flac`, `m4a`, `ogg`, `opus`).
- `bitrate` (text, default `192k`): Audio bitrate for lossy formats.
- `sampleRate` (select, default `keep`): Output sample rate in Hz (`keep`, `22050`, `44100`, `48000`, `96000`).
- `channels` (select, default `keep`): Channel configuration (`keep`, `mono`, `stereo`).
- `streamIndex` (text, default `0`): Stream index for multi-track audio files.

**Outputs:**
- `outputPath`: Path to extracted audio file.
- `format`: Output format.
- `bitrate`: Effective bitrate.
- `fileSize`: Output file size in bytes.

---

### 3. FFmpeg Create GIF (`ffmpeg_create_gif`)
Converts video segments into high-quality animated GIFs.

**Inputs:**
- `inputPath` (file, required): Source video file.
- `outputPath` (text): Output `.gif` destination path.
- `startTime` (text, default `00:00:00`): Start time in `HH:MM:SS` or seconds.
- `duration` (text, default `5`): Clip duration in seconds.
- `fps` (number, default `15`): Target frame rate.
- `width` (number, default `480`): Width in pixels (proportional aspect ratio maintained).
- `loopCount` (number, default `0`): `0` for infinite loop, or repeat count.
- `quality` (select, default `high`): `high` (2-pass dynamic palette generation) or `standard`.

**Outputs:**
- `outputPath`: Path to generated GIF.
- `fileSize`: Output GIF size in bytes.
- `fps`: Frame rate used.
- `width`: Image width in pixels.
- `duration`: Clip duration in seconds.

---

### 4. FFmpeg Cut Media Clip (`ffmpeg_cut_media`)
Trims audio or video files by start and end timestamps.

**Inputs:**
- `inputPath` (file, required): Source media file.
- `outputPath` (text): Destination clip path.
- `startTime` (text, required): Start position (`HH:MM:SS` or seconds).
- `endTime` (text): End position (`HH:MM:SS` or seconds).
- `duration` (text): Clip duration (used if `endTime` is blank).
- `mode` (select, default `fast_copy`): `fast_copy` (instant lossless stream copy) or `accurate_reencode` (frame-accurate re-encode).

**Outputs:**
- `outputPath`: Path to cut clip.
- `startTime`, `endTime`, `duration`: Applied slice boundaries.
- `mode`: Cut mode executed.
- `fileSize`: Output file size in bytes.

---

### 5. FFmpeg Probe Media Info (`ffmpeg_probe_media`)
Extracts comprehensive structural and technical metadata from any media file using `ffprobe`.

**Inputs:**
- `inputPath` (file, required): Source media file to probe.

**Outputs:**
- `format`: Container format name (e.g. `mp4`, `matroska`, `mp3`).
- `duration`: Duration in seconds (float).
- `fileSize`: File size in bytes.
- `bitrate`: Overall bitrate in bps.
- `videoCodec`: Codec name of primary video stream.
- `width`, `height`: Primary video resolution.
- `fps`: Primary video frame rate.
- `audioCodec`: Codec name of primary audio stream.
- `sampleRate`: Audio sample rate in Hz.
- `channels`: Audio channel count.
- `videoStreams`: List of all video stream descriptor objects.
- `audioStreams`: List of all audio stream descriptor objects.
- `tags`: Dictionary of metadata tags (title, artist, creation date, etc.).
- `rawJson`: Full raw JSON string output from `ffprobe`.

---

### 6. FFmpeg Generate Thumbnail (`ffmpeg_generate_thumbnail`)
Captures a single frame snapshot at a given timestamp.

**Inputs:**
- `inputPath` (file, required): Source video file.
- `outputPath` (text): Destination image path (`.jpg`, `.png`, `.webp`).
- `timestamp` (text, default `00:00:01`): Time offset to capture (`HH:MM:SS` or seconds).
- `format` (select, default `jpg`): Image format (`jpg`, `png`, `webp`).
- `width` (number, default `0`): Image width in pixels (0 for original).
- `height` (number, default `0`): Image height in pixels (0 for original).

**Outputs:**
- `outputPath`: Path to generated thumbnail image.
- `timestamp`: Snapshot timestamp.
- `fileSize`: Thumbnail file size in bytes.

---

### 7. FFmpeg Run Custom Filter (`ffmpeg_run_custom_filter`)
Applies custom FFmpeg filter graphs for advanced transformation pipelines.

**Inputs:**
- `inputPath` (textarea, required): Path to source file (one per line for multiple inputs in complex filters).
- `outputPath` (text, required): Destination path for filtered media.
- `filterType` (select, default `video_filter`): `video_filter` (`-vf`), `audio_filter` (`-af`), or `complex_filter` (`-filter_complex`).
- `filterGraph` (textarea, required): Filter expression (e.g. `scale=1280:720,eq=contrast=1.2`, `volume=1.5`).
- `extraArgs` (text): Additional FFmpeg encoder or mapping flags (e.g. `-c:v libx264 -crf 20 -c:a aac`).

**Outputs:**
- `outputPath`: Path to filtered file.
- `filterType`: Type of filter executed.
- `filterGraph`: Filter expression executed.
- `fileSize`: Output file size in bytes.

## Contract (v1.1.0)

**Install:** Extensions → **Install from file** → choose `ffmpeg-media-bridge.dsext`.

**Permissions:**
- **filesystem**
- **shell**

No example workflow: requires ffmpeg/ffprobe on PATH; not first-run friendly.

**Lint / scan / pack:**
```bash
python tools/deskstride_ext_cli.py check extensions/ffmpeg-media-bridge
python tools/deskstride_ext_cli.py pack extensions/ffmpeg-media-bridge -o ffmpeg-media-bridge.dsext
```

## Sidebar page

Sidebar **Media Inspector**: probe streams, codecs, and duration with ffprobe.
