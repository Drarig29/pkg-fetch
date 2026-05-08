# Patch Overview

- This repository is a binary-building fork for `DataDog/datadog-ci`, not a general-purpose replacement for upstream `@yao-pkg/pkg-fetch`.
- `DataDog/datadog-ci` consumes this fork through GitHub release artifacts, not through this package's NPM identity or runtime API.
- The compatibility contract is the release asset set and the accompanying SHA256 files consumed by the `DataDog/datadog-ci` standalone build.
- Release artifacts attached to this fork's GitHub releases are the source of truth for the binaries consumed by `DataDog/datadog-ci`.

## Version And Platform Matrix

- The active Node.js matrix is limited to Node.js 22 and Node.js 26.
- The active platform matrix includes Linux x64, Linux arm64, Windows x64, Windows arm64, macOS x64, macOS arm64, Alpine x64, and Alpine arm64.

## Binary Shape

- Binaries are built with `--with-intl=none` to minimize size.
- `Intl` is therefore unavailable at runtime unless the consumer supplies a polyfill or shim.
- `pkg-fetch` defaults to force-building so CI produces fork-owned artifacts instead of silently downloading upstream release assets.

## Patch Set

- Node.js 22 is kept in `patches.json` with no extra source patch.
- Node.js 26 carries the active source patch for Node.js SEA standalone payload injection and current compiler/toolchain compatibility.
- Older patch metadata is removed from the active matrix so `latest` and ranged resolution only select versions this fork actually builds.

## What's needed for Node 26

### Linux

Node 26 now requires `libatomic.so.1` at runtime, but it's not installed on default Debian/Ubuntu systems (see https://github.com/nodejs/node/issues/60790).

So we install the `libatomic` development package in the build Docker image and statically link `libatomic` in the Linux binaries.

### macOS

#### macOS Deployment Target and `std::atomic_ref`

`common.gypi` raises the macOS deployment target to 14.0 because V8's SIMD source uses `std::atomic_ref`. Apple's libc++ only exposes that API for a macOS 14.0+ deployment target; with Node's default 13.5 target, Apple Clang fails with `no member named 'atomic_ref' in namespace 'std'`.

#### Typed Array Base64 with Apple Clang

`builtins-typed-array.cc` routes base64 encode/decode calls through the non-atomic simdutf helpers. Apple Clang advertises `std::atomic_ref` support, which enables V8's SharedArrayBuffer atomic base64 path, but the bundled simdutf headers used by this Node.js build do not expose the expected atomic base64 declaration in that configuration. The build fails with `no member named 'atomic_binary_to_base64' in namespace 'simdutf'`, so this fork uses the regular simdutf base64 helpers for those builtins.

#### SEA Tail Payload to remove Mach-O surgery

`node_sea_bin.cc` avoids Node's default SEA injection path, which uses [LIEF](https://github.com/lief-project/LIEF) to rewrite the output executable. On macOS x64, that Mach-O rewrite can corrupt chained-fixup entries and produce a binary that segfaults at startup even after a successful `codesign`. The failure mode is tracked upstream in https://github.com/nodejs/node/issues/62893, explained in https://github.com/pnpm/pnpm/pull/11455, and Bitwarden hit the same wall and dropped Intel Mac CLI (https://github.com/bitwarden/directory-connector/pull/1077).

Instead of inserting a new Mach-O segment/resource, the patched build appends the SEA blob to the end of the executable, followed by an eight-byte little-endian blob size and a fixed sentinel.

At startup, the patched SEA lookup first preserves Node's existing postject resource path for binaries that still use it. If no postject resource is present, it reads the current executable tail, validates the sentinel and encoded size, and treats the appended blob as the single executable application payload.

The tail format is applied uniformly by this fork rather than only on macOS x64. The macOS x64 LIEF issue is the reason for avoiding the upstream injection path, but the resulting binaries work as-is with `DataDog/datadog-ci`'s end-to-end (E2E) standalone tests, so the fork keeps one tested payload strategy across platforms.

## CI And Build Reliability

- GitHub Actions build matrices are narrowed to the version and platform set used by `DataDog/datadog-ci`.
- macOS builds use newer macOS runners, explicit Clang selection, GNU make, compiler caching, a workspace-local build path, and no Link Time Optimization (LTO) to stay within GitHub Actions limits.
- Generated build and cache directories are ignored in Git.
