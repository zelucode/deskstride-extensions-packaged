# deskstride-extensions-packaged (public)

Built, signed DeskStride extension packages. **No source code lives here**, only artifacts produced by `tools/publish.py` in the
private `deskstride-extensions` repo. Do not edit by hand and never overwrite a published file: the registry pins each file's
sha256.

```
<extension-id>/<extension-id>-<version>.dsext       the package
<extension-id>/<extension-id>-<version>.dsext.sig   detached Ed25519 signature
```

Browse and install these from DeskStride (Settings → Extensions → Browse) or from the DeskStride Marketplace. The registry entries
that point here live in the `extensions-registry` repo.
