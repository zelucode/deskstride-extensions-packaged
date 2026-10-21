# Video Quality Sorter

Sorts the videos in a folder, sub-folders included, into **Low**, **Medium** and **High** quality folders.

Installing it adds a **Scan Videos by Quality** step and an example workflow, **Organize Videos by Quality**, to your Templates.

## What you need

FFmpeg installed. The step reads each video's size with `ffprobe`, which comes with FFmpeg.
If `ffprobe` is already available from your command line, nothing else is needed. Otherwise, set **FFmpeg folder** to the folder that contains it.

## How to use it

1. Open **Templates**, pick **Organize Videos by Quality**, and click Use Template.
2. Run it. Enter the **folder** to clean up. **Dry run** is on by default: you get a summary of what would move, and nothing is touched.
3. Run again with **Dry run** off to move the files.

By default each video goes into a `Low`, `Medium` or `High` folder **inside the folder it is already in**.
To collect every video of one quality in a single place instead, fill in **Low folder**, **Medium folder** and/or **High folder**.
A quality you leave blank still uses the default.

## How quality is decided

By the longer side of the picture, so portrait phone videos and wide-screen films are judged fairly:

| Quality | Longer side | Typical name |
|---|---|---|
| Low | under 1280 px | below 720p |
| Medium | 1280 to 1919 px | 720p up to just under 1080p |
| High | 1920 px or more | 1080p, 4K and above |

Both limits can be changed on the step.

## Good to know

- Running it again is safe. Files already in the right place are left alone.
- A file whose name already exists in the destination is kept and renamed, for example `clip (1).mp4`. Nothing is overwritten.
- Files that cannot be read as video are listed in the step's output and skipped. The run carries on.
- Only common video extensions are scanned. You can change the list on the step.
