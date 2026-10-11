# deskstride-extensions-packaged (public)

Built, signed DeskStride extension packages. This repo holds only release artifacts: the development repo is private. A `.dsext`
package is a plain zip archive, so its contents (including the extension's code) are readable by anyone. Do not edit by hand and
never overwrite a published file: the registry pins each file's sha256.

```
<extension-id>/<extension-id>-<version>.dsext       the package
<extension-id>/<extension-id>-<version>.dsext.sig   detached Ed25519 signature
<extension-id>/README.md                            the extension's README, shown on the marketplace page
```

`README.md` is the one file that may change without a new version: it is display-only (the marketplace renders it with all HTML
escaped) and is not covered by the registry's sha256. `tools/publish.py` in the source repo refreshes it on every run.

Browse and install these from DeskStride (Settings → Extensions → Browse) or from the DeskStride Marketplace. The registry entries
that point here live in the `extensions-registry` repo.

## Signer

All packages are signed with one Ed25519 release key:

| | |
|---|---|
| Fingerprint | `7AC6:6249:04F1:1BA1` |
| Public key (base64) | `fTd0CcgxxvNhXHB/I4mNS2XQ7102WP6P+NoSJ7eem0Q=` |

The `.sig` file next to each package contains the same public key. This key is built into DeskStride: **the app installs a package from
the official registry only if it is signed by it**, and refuses an unsigned one. It also blocks an update of an installed extension that is
signed by a different key. DeskStride checks the signature automatically when you install. A standalone command-line verifier is not published yet.

Rules for this repo: every `.dsext` needs its `.dsext.sig` next to it, made with the release key (a package signed with any other key is
refused by the app); files are added by `tools/publish.py` in the private source repo, never by hand, and never replaced or reused under the
same version; a package's dependencies must be exact pins. A checkout lets you re-check the whole repo:
`python tools/deskstride_ext_cli.py build-index <registry> --packages-root <this repo> --official` (run from the source repo).

A signature shows the package came from the holder of this key and was not altered. It does not make the code safe: extensions run
with the permissions they declare, so read what an extension asks for before installing it.
