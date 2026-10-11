# Component build and release format

Local, development and versioned builds use `scripts/build-components.py`.
`build-rootfs.sh` invokes the same component builder. CI produces the complete installable tarball; Install and Update consume it directly.

Actions caches each finished compressed archive independently. Cache hits skip
compilation, export and compression. Cache misses rebuild only the affected
parts, with BuildKit layer caching available for compiler work. Caches are
optional and disposable: an empty or evicted cache causes a normal rebuild.

Each release publishes one assembled system archive. CI validates every component
and the combined path/link graph, assembles the rootfs, and generates integration
metadata before compressing the final archive. The device verifies its download
checksum, extracts it once, and installs it; it does not assemble components or
generate integration archives. The final archive is a durable public download;
installers never depend on Actions caches or run artifacts. There are no individual
component releases or registry packages.

## Components and invalidation

| Component | Inputs that rebuild it |
| --- | --- |
| Guest base | Debian snapshot, runtime package recipe, Trash family, account/bootstrap policy |
| Trusted host runtime | Debian snapshot and host package recipe |
| Firefox media | Pinned sources, private FFmpeg build recipe, compiler snapshot |
| MPV media | Pinned sources, codec patches, compiler snapshot |
| Fuzzel | Launcher, Pixman and protocol sources/recipes, compiler snapshot |
| Keyboard | Sources, layout/patches, compiler snapshot |
| Xwayland | Satellite sources, compiler snapshot |
| Integration | Guest overlay, host helpers, installer, service/input files and GTK theme |

Trash packages are a ninth **build-only cache entry**, used to produce the guest
base. They are not an installed layer or a separately published download.

Changing Waybar CSS, a launcher flag or GTK theme rebuilds only integration. Changing an MPV patch rebuilds MPV media.
Changing the Trash family rebuilds its transaction payload and fresh-install
base. A base-only bootstrap change consumes the cached package artifacts instead
of rebuilding the native packages. Producer and archive ownership policy changes invalidate components conservatively.
Cache transport, release publication and CLI changes do not invalidate compiled
archives; guest/host helper changes rebuild integration.

Keys contain source bytes, modes, links, selected Docker stages, build architecture
where applicable, and the explicit dependency lock. Commit IDs, clocks and
per-run export attestations are excluded. Keys identify inputs; separate SHA-256
digests identify actual compressed artifact bytes. Existing bindings are immutable.
The source commit and timestamp are generated in CI and stored inside the tarball.

`build-support/components/dependencies.json` pins the Debian and Debian-security
snapshot used for component builds. Refresh it explicitly to adopt distribution
updates. The pinned base image alone does not freeze APT. Snapshot builds disable
only the historical index freshness deadline; signed indexes and package hashes
remain required. The guest base restores normal Debian update sources before
export; users' later APT security updates are not pinned to the build snapshot.
Private compiled components carry their source/license payloads.

Account creation and the randomly salted initial password run only in the stable
base target. Overlay changes do not rerun account initialization. Host theme
files are outside the host-runtime target. Fresh installs retain service owners,
hardlinks and setuid bits and use the existing one-time mapped ownership shift.

## Local commands

```sh
python3 scripts/build-components.py --plan
python3 scripts/build-components.py
```

Artifacts and immutable key bindings are in `build/component-store/`; the release
manifest, plan and timings are in `dist/components/`. Preserve the store between
local trials. No command above publishes or triggers a hosted build.

On a cross-host build, `ROCKNIX_TRASH_PACKAGES_DIR` can reuse an audited native
package directory. Its content is included in distinct local input keys and the
publisher refuses these local-override manifests. On hosts without `dpkg-deb`,
`ROCKNIX_PACKAGE_AUDITOR_IMAGE=sha256:...` may select an existing ARM64 Debian image
containing Python and dpkg to audit the package directory read-only without a
network. Normal Ubuntu ARM64 CI audits natively.

Assemble an install profile into a new staging directory, preserving ownership
under a single fakeroot session:

```sh
fakeroot -- python3 payload/bin/rocknix-components \
  --manifest dist/components/release.json --cache build/component-store \
  --profile install --output dist/component-install
```

Fakeroot ownership exists only for that session. Inspect or archive the result
inside the same fakeroot session; do not deploy its loose directory as a
metadata-preserving copy. The device installer extracts the completed archive as actual host root.

The same command with `--profile update` includes the same complete system.
The installer downloads only `rocknix-desktop-rp6-arm64.tar.xz` and its
`.sha256` file, verifies the checksum, and extracts the system with numeric owners
and permissions preserved. All build metadata is generated in CI and stored
inside the archive. No release API lookup, extra helper download, or device-side
metadata generation is needed. `packaging-timings.json` is a CI artifact, not a
release asset, and records assembly/validation and compression separately.

## Build artifact

The Build Desktop LXC workflow builds the pinned ARM64 components and uploads
the assembled tarball and SHA-256 file as a GitHub Actions artifact. It does not
publish a release or modify this repository. Runtime acceptance belongs to the
private development repository.

## Xwayland delivery

The Xwayland component carries the guest-owned Satellite bridge and its source
and license files. Debian X11 dependencies are installed in the guest base.
Install and Update both receive them in the new rootfs; there is no retained
container package updater or compatibility path for older component manifests without the bundled layout.
