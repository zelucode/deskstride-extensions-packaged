# Studio Themes

Three extra colour themes for DeskStride: **Dusk** (dark), **Sandstone** (light) and **Mint Light** (light).

This extension only changes how the app looks. It contains no code and asks for no permissions.

## Use it

1. Install it from the Extensions page.
2. Open Settings → Theme. The new themes appear next to the built-in ones, marked "from Studio Themes".
3. Pick one. Disabling or uninstalling the extension removes its themes; if one was active, the app falls back to a built-in theme.

## For authors

A theme-only extension needs just a `manifest.json` and one JSON file per theme under `themes/`:

```json
{
  "formatVersion": 1,
  "id": "my-themes",
  "name": "My Themes",
  "version": "1.0.0",
  "contributes": { "themes": ["themes/my-theme.json"] }
}
```

Each theme file needs `id`, `name`, `mode` (`dark` or `light`) and the colours `primaryColor`, `primaryHover`, `bgCanvas`, `cardBg`, `borderColor`, `textColor` (hex). Optional: `description`, `surfaceRaised`, `borderStrong`, `textSecondary`, `textMuted`, `primaryForeground`.

