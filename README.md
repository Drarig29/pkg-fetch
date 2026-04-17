## This project is a fork used by `DataDog/datadog-ci` (see [#1858](https://github.com/DataDog/datadog-ci/pull/1858))

This fork of https://github.com/yao-pkg/pkg-fetch has the following differences:

- It only builds Node.js binaries **required by `DataDog/datadog-ci`**:
  - Linux x64
  - Linux arm64
  - Windows x64
  - macOS x64
  - macOS arm64
  - Alpine x64
  - Alpine arm64
- It uses the `--with-intl=none`/`--without-intl` option to have a minimal size (see [No ICU support](#no-icu-support))
- It **does not** apply any patches to the Node.js binaries, so it **must be used with `@yao-pkg/pkg`'s `--sea` option**.

To avoid having to publish this fork on NPM (and also need to fork & publish `@yao-pkg/pkg`), this fork is only here to prebuild and hold the binaries for `DataDog/datadog-ci`.

We have Yarn patches in the `DataDog/datadog-ci` repository to adjust `@yao-pkg/pkg`'s behavior and change the expected checksums and download location in `@yao-pkg/pkg-fetch`.

The binaries are attached to [this release](https://github.com/Drarig29/pkg-fetch/releases/latest).

## No ICU support

Node.js binaries built by this project **use the `--with-intl=none` option** to drastically reduce the size of the binary.

- If you need a Node.js binary with small ICU support, you can build it yourself by setting the `--with-intl=small-icu` option. Small ICU is sufficient for most use cases.
- If you need a Node.js binary with full ICU support, you can build it yourself by setting the `--with-intl=full-icu` option.

Size comparison:

- Node.js 22 (macOS arm64) with `--with-intl=none` is 51.4 MB
- Node.js 22 (macOS arm64) with `--with-intl=small-icu` is 57.7 MB
- Node.js 22 (macOS arm64) with `--with-intl=full-icu` is 110 MB

Using `--with-intl=none` only has the following breaking changes:

- The `Intl` object **is not available**, so accessing it will throw a `ReferenceError`. To fix it, you can provide shims or install [Intl.js](https://github.com/andyearnshaw/Intl.js) as a polyfill.

See [this table in the Node.js documentation](https://nodejs.org/api/intl.html#options-for-building-nodejs) to see all the differences.
