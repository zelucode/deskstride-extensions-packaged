# deskstride-extensions-packaged (public)

Built, signed DeskStride extension packages. This repo holds only release artifacts: the development repo is private. A `.dsext`
package is a plain zip archive, so its contents (including the extension's code) are readable by anyone. Do not edit by hand and
never overwrite a published file: the registry pins each file's sha256.

```
<extension-id>/<extension-id>-<version>.dsext       the package
<extension-id>/<extension-id>-<version>.dsext.sig   detached Ed25519 signature
```

Browse and install these from DeskStride (Settings → Extensions → Browse) or from the DeskStride Marketplace. The registry entries
that point here live in the `extensions-registry` repo.

## Signer

All packages are signed with one Ed25519 release key:

| | |
|---|---|
| Fingerprint | `7AC6:6249:04F1:1BA1` |
| Public key (base64) | `fTd0CcgxxvNhXHB/I4mNS2XQ7102WP6P+NoSJ7eem0Q=` |

The `.sig` file next to each package contains the same public key. DeskStride shows the fingerprint when you install and warns if a
later update of the same extension is signed by a different key. DeskStride checks the signature automatically when you install. A standalone command-line verifier is not published yet.

A signature shows the package came from the holder of this key and was not altered. It does not make the code safe: extensions run
with the permissions they declare, so read what an extension asks for before installing it.
