## This project is a fork used by `DataDog/datadog-ci` (see [#1858](https://github.com/DataDog/datadog-ci/pull/1858) and [#2300](https://github.com/DataDog/datadog-ci/pull/2300))

This fork of https://github.com/yao-pkg/pkg-fetch has the following differences:

- It only builds Node.js binaries **required by `DataDog/datadog-ci`**:
  - Linux x64
  - Linux arm64
  - Windows x64
  - Windows arm64
  - macOS x64
  - macOS arm64
  - Alpine x64
  - Alpine arm64
- It uses the `--with-intl=none` (alias `--without-intl`) option to have a minimal size (see [No ICU support](#no-icu-support))
- It only tracks the Node.js versions currently needed by `DataDog/datadog-ci`: Node.js 22 and Node.js 26
- It keeps `pkg-fetch` in build mode by default, so GitHub Actions produces fork-owned binaries instead of downloading upstream release assets
- It publishes binaries as GitHub release assets for this fork, with SHA files generated from the workflow artifacts
- It carries the Node.js 26 patch needed for Node.js SEA standalone payload handling and current toolchain compatibility, while Node.js 22 builds without an extra source patch
- It tunes CI for the fork's release workflow, including newer macOS runners, macOS compiler caching, GNU make on macOS, reduced macOS build cost, and Linux `libatomic` handling for newer Node.js builds

See [PATCH.md](./PATCH.md) for an overview of the changes in this fork.

This fork is only here to prebuild and hold Node.js binaries for `DataDog/datadog-ci`; it is not published to NPM as a package consumed by `DataDog/datadog-ci`.

The `DataDog/datadog-ci` repository downloads custom Node.js binaries from this fork's GitHub releases and uses `node --build-sea` to create standalone binaries.

The binaries are attached to [this release](https://github.com/Drarig29/pkg-fetch/releases/latest).

## No ICU support

Node.js binaries built by this project **use the `--with-intl=none` option** to drastically reduce the size of the binary.

Size comparison:

- Node.js 22 (macOS arm64) with `--with-intl=full-icu` is 110 MB
- Node.js 22 (macOS arm64) with `--with-intl=small-icu` is 57.7 MB
- Node.js 22 (macOS arm64) with `--with-intl=none` is 51.4 MB

Using `--with-intl=none` only has the following breaking changes:

- The `Intl` object **is not available**, so accessing it will throw a `ReferenceError`. To fix it, you can provide shims or install [Intl.js](https://github.com/andyearnshaw/Intl.js) as a polyfill.

See [this table in the Node.js documentation](https://nodejs.org/api/intl.html#options-for-building-nodejs) to see all the differences.
