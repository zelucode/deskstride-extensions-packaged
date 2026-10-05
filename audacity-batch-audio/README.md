# Audacity Batch Audio

**Version:** 1.0.0 (Windows)

Normalize, apply effects to, and convert audio files by driving **Audacity** through its scripting pipe. Audacity must be installed
and `mod-script-pipe` enabled. No pip packages (the old pywin32 dependency is gone).

## Set up Audacity once

1. Install Audacity 3.x.
2. **Edit -> Preferences -> Modules** -> set **mod-script-pipe** to **Enabled**, then restart Audacity.
3. Optionally set **Audacity Executable Path** in this extension's settings so the nodes can start Audacity for you; otherwise start it yourself before running a workflow.

## Nodes

| Node | What it does |
|---|---|
| **Audacity Normalize** | Imports a file, normalizes the peak level (default -1 dB, optional DC-offset removal and independent stereo channels), exports a new file. |
| **Audacity Apply Effect** | One effect over the whole file, then export: `amplify`, `loudness-normalize` (LUFS), `fade-in`, `fade-out`, `low-pass`, `high-pass` (cutoff Hz + rolloff), `change-speed` (%), `reverse`. |
| **Audacity Convert Folder** | Converts every audio file in a folder to WAV / MP3 / FLAC / OGG / AIFF, optionally normalizing each; a file that fails is reported and the batch continues. |

All nodes write a **new** file (never the original), refuse to replace an existing output unless *Overwrite Existing Output* is on, set the
channel count explicitly (**Audacity's scripted export defaults to mono**, so stereo is requested unless you choose mono), and always remove
their imported tracks afterwards so the Audacity project does not fill up. Every step has a timeout, and Audacity going silent or quitting
mid-job produces a clear error instead of a hang.

## What is not here, and why

**Noise Reduction is not offered.** Audacity's own scripting reference states it is "not currently available from scripting". The
earlier draft of this extension had a Noise Reduction node that could never have worked.

## Settings

- **Audacity Executable Path**: optional; used to start Audacity when its pipes are not found.
- **Default Output Folder**: where results go when a node's output path is blank (otherwise next to the input).

## Contract (v1.0.0)

**Install:** Extensions -> **Install from file** -> choose `audacity-batch-audio-1.0.0.dsext`.

**Permissions:**
- **filesystem**
- **shell** (may start Audacity)

**Platform:** Windows (uses Audacity's Windows named pipes `\\.\pipe\ToSrvPipe` and `\\.\pipe\FromSrvPipe`).

**Verified with:** `tests/extensions/test_audacity.py`, which runs every node against a stand-in that speaks Audacity's documented pipe protocol over real OS pipes (command sequences, quoting, failures, silence, disconnects, cleanup).
The command and parameter names were checked against Audacity's Scripting Reference. **It has not been run against a real Audacity** (not installed on the build machine), so try it once on a short file before relying on it.
