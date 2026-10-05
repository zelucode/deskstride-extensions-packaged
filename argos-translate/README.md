# Argos Translate Extension

**Version:** 1.0.4

Offline neural machine translation using [Argos Translate](https://github.com/argosopentech/argos-translate). Translate text between languages using locally installed models. All processing happens on-device — no network requests during translation.

## Features

- **Offline translation** — No internet required after model installation
- **5 node types** — Translate single text, batch translate, list languages, install/remove models
- **Local models** — Models stored in configurable directory (Argos default: `~/.local/share/argos-translate/packages` on Linux/macOS, `%LOCALAPPDATA%\argos-translate\packages` on Windows)
- **Model Manager page** — Sidebar page (Extensions → Argos Translate) to see installed models, download a language pair before running a workflow, remove one, and refresh the list of available pairs
- **CPU & CUDA support** — Auto-detect or explicit device selection
- **Multiple SBD options** — MiniSBD (default, fast), Stanza, or spaCy
- **Private dependencies** — `argostranslate==1.11.0` installed in extension's isolated `deps/` folder

## Installation

1. Open **Extensions** → **Install from file**
2. Select `argos-translate.dsext`
3. Confirm the install prompt (dependencies install into this extension's private folder)
4. Nodes appear in the palette under **Translate**

## Permissions

- **network** — fetching the model index and downloading language-pair packages
- **filesystem** — reading/writing models under the configured model directory

## Example workflow

On install, **Argos Translate Demo** is added to Workflows (list languages, install a pair, translate, cleanup).

## Model Management

**Models are not bundled.** You must download and install them using the provided nodes:

1. **List available packages**: Use `Argos List Languages` with `refreshAvailable=true` to fetch the remote index (requires network)
2. **Install a model**: Use `Argos Install Model` with `fromCode`/`toCode` (e.g., `en` → `es`)
3. **Translate**: Use `Argos Translate Text` or `Argos Translate Batch` with installed language codes
4. **Remove a model**: Use `Argos Remove Model` with `fromCode`/`toCode`

### Language Codes

Argos uses ISO 639-1 codes (e.g., `en`, `es`, `fr`, `de`, `zh`, `ja`, `ko`, `ru`, `ar`, `hi`, `pt`, `it`, `nl`, `pl`, `tr`, `vi`, `th`, `id`, `sv`, `da`, `no`, `fi`, `cs`, `sk`, `hu`, `ro`, `bg`, `hr`, `sr`, `sl`, `et`, `lv`, `lt`, `uk`, `be`, `mk`, `sq`, `mt`, `ga`, `cy`, `eu`, `ca`, `gl`, `is`, `fo`, `kl`, `mi`, `sm`, `to`, `fj`, `haw`, `mg`, `rn`, `sg`, `sn`, `so`, `st`, `sw`, `ts`, `tn`, `ve`, `xh`, `zu`).

Run `Argos List Languages` with `refreshAvailable=true` to see all available pairs.

## Settings

Configure via Extensions page → Argos Translate → Settings:

| `modelDir` | text | *(Argos default)* | Custom model directory. Blank = Argos default (`~/.local/share/argos-translate/packages` on Linux/macOS, `%LOCALAPPDATA%\argos-translate\packages` on Windows) |
| `device` | select | `auto` | `auto` (CUDA only if a GPU **and** the CUDA 12 runtime are present, else CPU), `cpu`, `cuda` (falls back to CPU with a log line if the CUDA runtime is missing) |
| `packageIndex` | text | *(Argos default)* | Custom package index URL |
| `sentenceBoundaryDetection` | select | `minisbd` | `minisbd` (fast), `stanza` (requires stanza), `spacy` (requires spaCy) |

Settings map to `ARGOS_*` environment variables set before each Argos import.

## Node Contracts

### `argos_translate_text`
Translate a single text between languages.

**Inputs:**
- `fromCode` (text, required) — Source language code (e.g., `en`)
- `toCode` (text, required) — Target language code (e.g., `es`)
- `text` (textarea) — Text to translate. Paragraphs/newlines preserved.

**Outputs:**
- `translatedText` (string) — Translated text
- `fromCode` (string) — Source code
- `toCode` (string) — Target code
- `inputLength` (number) — Input character count
- `outputLength` (number) — Output character count

**Errors:**
- `fromCode`/`toCode` required
- Model not installed for pair
- Translation failure

### `argos_translate_batch`
Translate multiple text items (one per line).

**Inputs:**
- `fromCode` (text, required) — Source language code
- `toCode` (text, required) — Target language code
- `text` (textarea) — One text item per line. Blank lines skipped.

**Outputs:**
- `translatedText` (string) — Joined translations (newline-separated)
- `items` (array) — Array of `{index, original, translated}`
- `fromCode`, `toCode` (string) — Language codes
- `count` (number) — Number of translated items

### `argos_list_languages`
List installed languages/packages. Optionally refresh remote index.

**Inputs:**
- `refreshAvailable` (boolean) — Fetch remote index (requires network). Default: false.

**Outputs:**
- `installedLanguages` (array) — `{code, name}`
- `installedPackages` (array) — `{fromCode, toCode, fromName, toName, path}`
- `availablePackages` (array) — `{fromCode, toCode, fromName, toName, packageVersion, argosVersion}`
- `modelDirectory` (string) — Effective model directory path

### `argos_install_model`
Install a direct language-pair model.

**Inputs:**
- `fromCode` (text, required) — Source language code
- `toCode` (text, required) — Target language code
- `refreshIndex` (boolean) — Refresh index first. Default: false.

**Outputs:**
- `fromCode`, `toCode` (string)
- `fromName`, `toName` (string)
- `packageVersion`, `argosVersion` (string)
- `installedPath` (string) — Path to installed model

### `argos_remove_model`
Remove an installed direct package.

**Inputs:**
- `fromCode` (text, required) — Source language code
- `toCode` (text, required) — Target language code

**Outputs:**
- `fromCode`, `toCode` (string)
- `removed` (boolean) — Always true on success

**Errors:**
- Package not found (lists installed pairs)

## Performance & Memory

- **Model size**: ~30-100 MB per language pair (varies by language)
- **RAM usage**: Model loaded into memory on first translation (~200-500 MB per model)
- **First translation**: Slower (model load + warmup)
- **Subsequent translations**: Fast (~10-50 ms per sentence on CPU)
- **CUDA**: Faster for batch/long text, but needs the CUDA 12 runtime (cuBLAS + cuDNN 9) installed — a GPU driver alone is not enough. Without it the extension uses CPU and logs why. CPU handles a short paragraph in about a second after a ~3 s one-time import.
- **Concurrent**: Multiple translations share loaded models (Argos caches internally)

## Windows / Python Constraints

- **Python 3.9+** required (Argos Translate dependency)
- **No sandbox** — Extension runs in-process with full privileges
- **Thread-safe** — Global lock around Argos state (`_argos_lock`)
- **Lazy imports** — Argos imported only when node executes, not at registry load
- **Private deps** — `argostranslate` installed in `deps/`, not system Python
- **Model directory** — Must be writable; default `%LOCALAPPDATA%\argos-translate\packages` on Windows

## Development

### Build & Test

```bash
python tools/deskstride_ext_cli.py lint extensions/argos-translate
python tools/deskstride_ext_cli.py scan extensions/argos-translate
python tools/deskstride_ext_cli.py check extensions/argos-translate
python tools/deskstride_ext_cli.py pack extensions/argos-translate -o argos-translate.dsext
```

Run tests (requires pytest):

```bash
cd workspace/argos-translate
python -m pytest tests/test_argos_translate.py -v
```

### Test Design

Tests are **offline and fast** — they mock `argostranslate` modules and registry.
No real models are downloaded. Tests cover:
- Input validation (missing codes, empty text)
- Missing dependency handling
- Missing model handling
- Language listing (installed + available)
- Install/remove behavior
- Translation delegation
- Batch translation with blank-line skipping

## Architecture

```
argos-translate/
├── manifest.json
├── requirements.txt
├── nodes/
│   ├── __init__.py
│   ├── _argos_utils.py       # Shared: lazy imports, env config, lock, helpers
│   ├── argos_translate_text.py
│   ├── argos_translate_batch.py
│   ├── argos_list_languages.py
│   ├── argos_install_model.py
│   └── argos_remove_model.py
├── tests/
│   └── test_argos_translate.py
└── README.md
```

- `_argos_utils.py` is **not** in `nodeModules` — imported as sibling by node modules
- All Argos imports are lazy (inside functions/context managers)
- `threading.RLock` protects Argos global state across concurrent node runs
- Environment variables set per-call via `argos_env(ctx)` context manager

## Troubleshooting

| Issue | Solution |
|-------|----------|
| `ModuleNotFoundError: argostranslate` | Reinstall extension (triggers pip install) |
| `Model not installed for en -> es` | Run `Argos Install Model` with `fromCode=en`, `toCode=es` |
| `CUDA out of memory` | Set `device=cpu` in settings or use smaller batch |
| `Library cublas64_12.dll is not found` | CUDA runtime not installed. Current versions fall back to CPU automatically; install the CUDA 12 runtime (cuBLAS + cuDNN 9) if you want the GPU, or set `device=cpu` |
| `Permission denied` on model dir | Set `modelDir` to writable path in settings |
| Slow first translation | Expected — model loads on first use |

## License

Argos Translate is licensed under MIT. This extension follows the same license.