# Website Watcher — example extension

**Version:** 1.1.0

Adds **Watch Website**, a node that fetches a URL (optionally scoped to a
CSS selector) and reports whether its content changed since the last run.
Built for `integration-lab/BACKLOG.md` item 2.

## Permissions

| Permission | Why |
|---|---|
| `network` | Fetches the URL you ask it to watch |

## Strategy: B (port the design, not the code)

Huginn's own source is MIT-licensed, but it's Ruby — `Strategy B` per this
lab's convention means reading its **Website Agent** for *design* (what
state it tracks between runs, how it decides "new" vs "seen") and writing
fresh code, not translating or copying its code. `LICENSE_NOTES.md`'s
Huginn entry (checked 2026-09-03) already flags exactly this: MIT but Ruby,
port logic not code.

The two other Huginn agents named in this backlog item are deliberately
**not** built here:

- **RSS Agent** — `BACKLOG.md` item 8 separately covers RSS/Atom polling via
  the `feedparser` package, explicitly informed by this item's Huginn
  research. Building RSS logic here would just be redone there.
- **Weather Agent** — a thin wrapper around one specific weather API with
  its own auth key. It doesn't match this item's actual "why it matters" (a
  recurring watch-and-detect-change shape with no dedicated node today) —
  it's a one-off integration, not the pattern. Skipped; revisit only if a
  concrete need for it shows up.

## The blocker this item surfaced, and how it got resolved

Huginn's Website Agent works by storing "what did I see last time" between
runs. When this item was scoped, this app's extension nodes had **no**
persistent state store — `ctx` only exposed settings (`ctx.ext`), and
`set_variable`'s `persist: true` is a core *workflow* node with a single
shared namespace, not something scoped per node instance an extension could
use safely (two Watch Website nodes on two different URLs would collide).

That gap is now closed: `ctx.node_state` is a per-`(workflow, node instance)`
key-value store, persisted by the app itself between runs. See
`reference-docs/creating-extensions.md`'s "Node state" section for the full
API. This node is the first thing in this lab built against it — no
self-managed state file, no extra settings field for a state directory.

## Try it

1. Sidebar → **Extensions** → **Install from file...** → pick
   `website-watcher.dsext`.
2. Watch the streamed dependency install log (`beautifulsoup4`).
3. Drag **Watch Website** onto a workflow. Set **URL**, and optionally a
   **CSS Selector** to watch just part of the page (e.g. `.price` for a
   product price, `#status` for a status banner) instead of the whole
   page's text.
4. Run it once — this is the baseline. `changed` is `false` and `firstRun`
   is `true`, since there's nothing to compare against yet.
5. Run it again after the page (or the selected part of it) actually
   changes — `changed` becomes `true`, with `currentContent` and
   `previousContent` both in the output.
6. Wire a **Condition** node on `changed` into whatever notify step you
   already have — this node's job stops at detecting and reporting the
   change.
7. Open the node's **Properties panel** → **Node state** section to inspect
   or **Clear** its stored snapshot (e.g. to force the next run to be
   treated as a fresh baseline).

## Example workflow

After install, open **Example: Watch Website**. It watches `example.com`'s
`h1` and branches on whether content changed. First run establishes the
baseline; re-run later to compare. No secrets required.

## What the node does

**Watch Website** (`watch_website`):
- Fetches the URL and extracts visible text (optionally scoped to a CSS
  selector). Raises an input error if a selector matches nothing.
- Hashes the extracted text and compares it against the last run's stored
  snapshot (`ctx.node_state`).
- Truncates what it stores/hashes to 20,000 characters so state stays
  bounded.
- Returns `changed` (bool), `firstRun` (bool — true only on the very first
  run for this node instance, so a workflow can skip notifying on the
  initial baseline), `currentContent`, `previousContent`.

## What this extension deliberately doesn't do

- **No diff algorithm.** `previousContent`/`currentContent` are both
  returned as plain text; producing a human-readable diff (or an LLM
  summary of what changed) is left to a downstream node — this node's job
  is detection, not summarization.
- **No built-in notification.** See step 6 above.
- **No JS rendering.** This fetches the HTML response as returned by the
  server — a page whose watched content is only rendered client-side won't
  be visible to it. Use browser automation nodes for that case.

## Package it

```bash
python tools/deskstride_ext_cli.py lint extensions/website-watcher
python tools/deskstride_ext_cli.py scan extensions/website-watcher
python tools/deskstride_ext_cli.py pack extensions/website-watcher -o website-watcher.dsext
```
