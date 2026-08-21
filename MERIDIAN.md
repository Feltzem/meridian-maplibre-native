# Meridian MapLibre Native fork

This repository is **Meridian's pinned fork of MapLibre Native**, created for
the 3D terrain slice of the Meridian GIS app (iPadOS + native macOS). It exists
to turn the upstream `feature/terrain-3d` work into a supportable, reproducible
Apple dependency — see `docs/v1.2-raster-terrain-completion-plan.md` in the
Meridian repository for the full plan, phases, and gates.

## Branch policy

- **`meridian-terrain`** (default) is the working branch. It is upstream
  `feature/terrain-3d` at the recorded base commit plus a small set of carried
  Meridian patches.
- **Base commit:** `f51c4e0ecc03e1b24244b8b89e905aaeb56004ba`
  (upstream `feature/terrain-3d` HEAD as of 2026-08-20, which was also the
  audit point named in the Meridian plan — no upstream drift existed when the
  fork was cut on 2026-08-21).
- Never rebase casually: rebases happen only at scheduled qualification points,
  and the new base commit is recorded here when they do.
- Every carried patch must be a small, reviewable commit with an upstream
  issue/PR link in its message. Patches that upstream accepts get dropped at
  the next rebase.

## Patch ledger

| Commit | Purpose | Upstream link | Status |
|---|---|---|---|
| (repo scaffolding) | `MERIDIAN.md` + `meridian-apple-release` workflow | n/a (Meridian-only) | permanent |

_No renderer patches are carried yet. Phase 2 work (Apple terrain API,
camera anchoring, depth picking, Metal fixes) lands here as it happens._

## Building the Apple XCFrameworks

Reproducible builds run in CI: the **`meridian-apple-release`** workflow
(`.github/workflows/meridian-apple-release.yml`, `workflow_dispatch`) builds

- the **iOS dynamic XCFramework** (device + simulator) via
  `bazel build --compilation_mode=opt --features=dead_strip,thin_lto
  --objc_enable_binary_stripping --apple_generate_dsym --output_groups=+dsyms
  --//:renderer=metal //platform/ios:MapLibre.dynamic`, and
- the **native macOS dynamic XCFramework** via the same flags plus
  `--//platform/macos:macos_loop=cfrunloop //platform/macos:MapLibre.dynamic`
  (with `brew install webp libuv`),

exactly matching upstream's own `ios-release.yml` / `macos-release.yml`
recipes at the base commit, then publishes dSYMs, SHA-256 checksums, and build
metadata (source commit, flags) as artifacts — and as a GitHub release
`apple-v<version>` when asked to.

The same commands work locally on an Apple-silicon Mac with Bazel (`.bazelversion`
pins the version; run from `platform/ios` / `platform/macos`).

## Distribution

Meridian apps never consume this repository directly. Checksummed release
assets are wrapped by
[`meridian-maplibre-apple-distribution`](https://github.com/Feltzem/meridian-maplibre-apple-distribution),
a Swift Package manifest pinned by semantic tag + checksum, which is what
`project.yml` / `project-mac.yml` point at. Never point the app at a moving
branch.

## Upstream terrain status at the base commit

Upstream's `TERRAIN.md` (this repo, root) is the authoritative status document.
Highlights as of the base commit: terrain renders on all four backends;
layer draping, elevated symbols/circles/fill-extrusions (Metal fixed
2026-08-02), skirts, elevation-aware tile cover, and CPU elevation queries are
implemented; development/testing has been Android/OpenGL-first, with Vulkan
device-verified and **Metal far less exercised** — which is exactly what
Meridian's Phase 0/2 qualification must cover. There is **no Apple
(`MLN*`) terrain API yet**: terrain is reachable only through the style JSON
`terrain` root, which is sufficient for the feasibility harness; a minimal
ObjC/Swift API (set/update/remove terrain, query configuration, elevation at
coordinate) is planned Phase 0/2 work in this fork.

## Licence

MapLibre Native is BSD-2-Clause (see `LICENSE.md` and `LICENSES.core.md`).
Meridian's carried patches are contributed under the same licence.
