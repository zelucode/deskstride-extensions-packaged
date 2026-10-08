# Media Download (yt-dlp) — example extension

**Version:** 2.1.1

Adds two node types backed by yt-dlp's own package (bundled via
`pipDependencies`): **Download Media (yt-dlp)** and **Get Media Info
(yt-dlp)**, plus a **Media Info** sidebar page. Built for `integration-lab/BACKLOG.md` item 5.

## Permissions

| Permission | Why |
|---|---|
| `network` | Fetches media/metadata from the URL you give it |
| `filesystem` | Writes downloads to the output folder you choose |
| `shell` | May launch ffmpeg (when configured) for merges / audio extract |

## Strategy: A (pip dependency), reversed from an earlier D (bridge)

`LICENSE_NOTES.md`'s yt-dlp entry (checked 2026-09-03, re-verified live
2026-09-04 via GitHub's license API — still **The Unlicense**, unchanged)
found this to be the single safest project surveyed in this lab: the
Unlicense is a public-domain dedication with no conditions at all, so
either strategy is legally clean — this was never a licensing decision.

**v1.x shelled out to a separately-installed `yt-dlp` CLI (Strategy D)**, the
same external-process relationship this app has with n8n/Telegram, on the
theory that a system-installed CLI updates on the user's own schedule (yt-dlp
ships frequent point releases specifically to fix site-extraction breakage)
while a `pipDependencies` pin would freeze a version until this extension
itself is updated and reinstalled. That tradeoff was real, but a second,
more fundamental problem surfaced during progress-reporting work (see
`docs/roadmap/RND_YTDLP_EXTENSION_PROGRESS_DEBUGGING.md` in the main repo,
its "actual open question" section) and forced a reversal: **yt-dlp's
official Windows release binary (a frozen/PyInstaller build) fully buffers
its own stdout the instant it isn't attached to a real console** — exactly
what piping through `subprocess.Popen` always means — and this held true
even with `PYTHONUNBUFFERED=1` set on the child process (confirmed by
directly reproducing it: piping the exact same command to a file outside
the app still withheld every progress line until the process exited). No
amount of subprocess-reading cleverness can produce progress bytes the
child never wrote to the pipe in the first place, so live progress
reporting was **not achievable at all** under the CLI-bridge approach on
Windows, no matter how `stream_subprocess`/the marker-line parsing was
written.

**v2.0 (this version) installs `yt-dlp` via `pipDependencies` into this
extension's private `deps/` folder instead**, and calls its Python API
(`YoutubeDL(...).extract_info(...)`) in-process. `progress_hooks` is a
plain Python callback invoked directly by yt-dlp's own downloader with real
numeric `downloaded_bytes`/`total_bytes` — no subprocess, no pipe, no
buffering to fight, no text-marker parsing. This is the accepted tradeoff:
the extension itself now owns bumping the pinned `yt-dlp` version to keep up
with site-extraction changes (was previously the end user's job via their
own CLI updates), in exchange for progress reporting actually working at
all. See the ffmpeg note directly below for the one piece that's still an
external dependency either way.

**Honest limitation carried over from v1.x:** yt-dlp's own site-extraction
logic changes as sites change their pages; the pinned version in
`manifest.json` (currently matching what was confirmed working during this
rewrite) will need bumping over time the same way any other pinned
dependency does.

**ffmpeg is a second, separate external dependency — install it too.**
yt-dlp shells out to `ffmpeg` for anything beyond the simplest single-file
download: merging separately-fetched best-video/best-audio streams (the
*default* behavior for "best quality," not an edge case) and this
extension's own **Audio Only** option (the `FFmpegExtractAudio`
postprocessor, which is a post-download conversion). Without `ffmpeg` on
`PATH` (or set via the
**ffmpeg Path** setting below), those cases fail or silently fall back to a
lower-quality pre-merged format — install it separately
(https://ffmpeg.org/download.html, or your OS package manager) and confirm
`ffmpeg -version` works in a terminal. **Not required at all** for
**Get Media Info** or a plain single-file download.

## Try it

1. Install `ffmpeg` separately and confirm `ffmpeg -version` works in a
   terminal — required for merged best-quality downloads and the Audio Only
   option (see "ffmpeg is a second, separate external dependency" above).
   yt-dlp itself no longer needs a separate install; it's pulled in
   automatically via `pipDependencies` on install.
2. Sidebar → **Extensions** → **Install from file...** → pick `yt-dlp-media-download.dsext`.
3. Set **ffmpeg Path** on the extension's settings card if `ffmpeg` isn't on
   PATH and you plan to use best-quality merges or Audio Only.
4. Optionally set **Default Output Folder**.
5. **Download Media (yt-dlp)** and **Get Media Info (yt-dlp)** appear in the
   palette's "http" category.
6. Sidebar → **Extensions** group → **Media Info** — paste a URL to see
   title/duration/thumbnail without downloading.

No example workflow ships with this extension — a realistic demo needs a live media URL (and often ffmpeg), which is not a safe first-run default.

## What each node does

- **Download Media (yt-dlp)** (`download_media`): downloads a URL's
  media (`outtmpl` filename template under Output Folder), optionally
  audio-only (`FFmpegExtractAudio` postprocessor, mp3), optionally with an
  advanced format selector. Defaults to `noplaylist: true` so pointing it at
  a playlist URL downloads only the one linked item, not the whole playlist
  — override explicitly if you want playlist behavior. Registers an
  `add_post_hook` callback to reliably get the final on-disk path *after*
  any post-processing (e.g. audio conversion) moves/renames the file — the
  Python-API equivalent of the old CLI's `--print after_move:filepath`.
  Reports live percent/speed/ETA through `ctx.emit_progress`, driven
  directly by yt-dlp's own `progress_hooks` callback (real
  `downloaded_bytes`/`total_bytes` integers, no text parsing) plus a
  `postprocessor_hooks` callback for the merge/audio-conversion phase, which
  has no percent of its own. See `_ytdlp.py`'s module docstring for why this
  replaced an earlier CLI-bridge (`node_bridge.stream_subprocess` +
  `--progress-template`) version: that approach could never report live
  progress at all on Windows, because yt-dlp's official release binary
  fully buffers its own stdout once piped, regardless of `PYTHONUNBUFFERED`.
- **Get Media Info (yt-dlp)** (`get_media_info`): `extract_info(...,
  download=False)` — fetches metadata only (title, uploader, duration,
  thumbnail, id, extractor) plus the full raw JSON (via yt-dlp's own
  `sanitize_info`) as a string output for anything not already broken out.

Both nodes raise a clear `NodeInputError` (shown as a user-mistake in the Run
Audit view) for a malformed URL, and a `RuntimeError` wrapping yt-dlp's own
`DownloadError` message if the download/lookup itself fails (bad URL,
extractor error, network failure, etc.) — yt-dlp's own error message is
almost always more specific than anything this wrapper could add.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/yt-dlp-media-download
python tools/deskstride_ext_cli.py scan extensions/yt-dlp-media-download
python tools/deskstride_ext_cli.py pack extensions/yt-dlp-media-download -o yt-dlp-media-download.dsext
```
