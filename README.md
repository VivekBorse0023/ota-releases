# ota-releases

Public artifact host for OTA updates across multiple embedded projects.
Source stays in each project's own private repo; this repo holds only
compiled, signed release artifacts and version manifests, so devices in
the field can check for and download updates over plain HTTPS with no
credentials -- private-repo Release assets require a token even just to
download, this repo doesn't.

## Convention

**Release tags**: `<project>-<component>-v<version>`, e.g.:
- `horiba-app-v0.1.0` -- app-only update (binaries + config, no rootfs swap)
- `horiba-full-v0.1.0` -- full image update (A/B rootfs swap)

**Manifests**: `manifests/<project>-<component>/latest.json`, e.g.
`manifests/horiba-app/latest.json`:

```json
{
  "version": "v0.1.0",
  "tag": "horiba-app-v0.1.0",
  "url": "https://github.com/VivekBorse0023/ota-releases/releases/download/horiba-app-v0.1.0/horiba-app-v0.1.0.swu",
  "sha256": "<checksum of the .swu file>"
}
```

A device polls its manifest via `raw.githubusercontent.com` (a static
file fetch through GitHub's CDN -- no auth, no API rate limit, unlike
hammering the REST API from many field devices), compares `version`
against what's installed, and if newer, downloads `url` and verifies it
against `sha256` before handing it to swupdate. The `.swu` bundle itself
is also signature-verified by swupdate independently of this checksum --
the checksum here just confirms a clean download, it isn't the trust
boundary.

## Adding a new project

Pick a `<project>` prefix, publish releases under `<project>-<component>-vX.Y.Z`
tags with the `.swu` (or other artifact) attached, and add/update
`manifests/<project>-<component>/latest.json` pointing at it. No other
setup needed in this repo.
