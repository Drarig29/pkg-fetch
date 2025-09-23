## This project is a fork used by `DataDog/datadog-ci` (see [#1858](https://github.com/DataDog/datadog-ci/pull/1858))

This fork has the following differences:

- It only builds Node.js binaries **required by `DataDog/datadog-ci`** (see [commit](https://github.com/Drarig29/pkg-fetch/commit/f773148f7e1d7c79fbec1ce5683368ce10016278)):
  - Linux x64
  - Linux arm64
  - Windows x64
  - macOS x64
  - macOS arm64
- It uses the `--with-intl=none` option to have a minimal size (see [No ICU support](#no-icu-support))
- It **does not** apply any patches to the Node.js binaries, so it **must be used with `@yao-pkg/pkg`'s `--sea` option**.

To avoid having to publish this fork on NPM (and also need to fork & publish `@yao-pkg/pkg`), this fork is only here to prebuild and hold the binaries for `DataDog/datadog-ci`.

We have Yarn patches in the `DataDog/datadog-ci` repository to adjust `@yao-pkg/pkg`'s behavior and change the expected checksums and download location in `@yao-pkg/pkg-fetch`.

The binaries are attached to [this release](https://github.com/Drarig29/pkg-fetch/releases/latest).

## Binary Compatibility

| Node                                                                                                                              | Platform | Architectures             | Minimum OS version                                                                  |
| --------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------- | ----------------------------------------------------------------------------------- |
| 8<sup>[1](#fn1)</sup>, 10<sup>[1](#fn1)</sup>, 12<sup>[1](#fn1)</sup>, 14<sup>[1](#fn1)</sup>, 16<sup>[1](#fn1)</sup>             | linux    | x64                       | Enterprise Linux 7, Ubuntu 14.04, Debian jessie, other distros with glibc >= 2.17   |
| 18, 20, 22                                                                                                                        | linux    | x64                       | Enterprise Linux 8, Ubuntu 20.04, Debian buster, other distros with glibc >= 2.28   |
| 8<sup>[1](#fn1)</sup>, 10<sup>[1](#fn1)</sup>, 12<sup>[1](#fn1)</sup>, 14<sup>[1](#fn1)</sup>, 16<sup>[1](#fn1)</sup>             | linux    | arm64                     | Enterprise Linux 8, Ubuntu 18.04, Debian buster, other distros with glibc >= 2.27   |
| 18, 20, 22                                                                                                                        | linux    | arm64                     | Enterprise Linux 9, Ubuntu 20.04, Debian bullseye, other distros with glibc >= 2.31 |
| 8<sup>[1](#fn1)</sup>, 10<sup>[1](#fn1)</sup>, 12<sup>[1](#fn1)</sup>, 14<sup>[1](#fn1)</sup>, 16<sup>[1](#fn1)</sup>, 18, 20, 22 | macos    | x64                       | 10.13                                                                               |
| 14<sup>[1](#fn1)</sup>, 16<sup>[1](#fn1)</sup>, 18, 20, 22                                                                        | macos    | arm64<sup>[2](#fn2)</sup> | 11.0                                                                                |
| 8<sup>[1](#fn1)</sup>, 10<sup>[1](#fn1)</sup>, 12<sup>[1](#fn1)</sup>, 14<sup>[1](#fn1)</sup>, 16<sup>[1](#fn1)</sup>, 18, 20, 22 | win      | x64                       | 8.1                                                                                 |

<em id="fn1">[1]</em>: end-of-life, may be removed in the next major release.

<em id="fn2">[2]</em>: [mandatory code signing](https://developer.apple.com/documentation/macos-release-notes/macos-big-sur-11_0_1-universal-apps-release-notes) is enforced by Apple.

## Security

We do not expect this project to have vulnerabilities of its own. Nonetheless, as this project distributes prebuilt Node.js binaries,

**Node.js security vulnerabilities affect binaries distributed by this project, as well.**

Like most of you, this project does not have access to advance/private disclosures of Node.js security vulnerabilities. We can only closely monitor the **public** security advisories from the Node.js team. It takes time to build and release a new set of binaries, once a new Node.js version has been released.

**It is possible for this project to fall victim to a supply chain attack.**

This project deploys multiple defense measures to ensure that the safe binaries are delivered to users:

- Binaries are compiled by [Github Actions](https://github.com/yao-pkg/pkg-fetch/actions)
  - Workflows and build logs are transparent and auditable.
  - Artifacts are the source of truth. Even repository/organization administrators can't tamper them.
- Hashes of binaries are hardcoded in [source](https://github.com/yao-pkg/pkg-fetch/blob/HEAD/lib/expected.ts)
  - Origins of the binaries are documented.
  - Changes to the binaries are logged by VCS (Git) and are publicly visible.
  - `pkg-fetch` rejects the binary if it does not match the hardcoded hash.
- GPG-signed hashes are available in [Releases](https://github.com/yao-pkg/pkg-fetch/releases)
  - Easy to spot a compromise.
- `pkg-fetch` package on npm is strictly permission-controlled

## Building a Binary Locally

You can use the `yarn start` script to build the binary locally, which is helpful
when updating patches to ensure functionality before pushing patch updates for
review.

For example:

`yarn start --node-range node18 --arch x64 --output dist`

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

## Environment

| Var              | Description                                                                                                  |
| ---------------- | ------------------------------------------------------------------------------------------------------------ |
| `PKG_BUILD_PATH` | Directory to use to clone and build nodejs binaries. Default to system temporary directory                   |
| `PKG_CACHE_PATH` | Path to pkg-cache. Default to `~/.pkg-cache`                                                                 |
| `PKG_IGNORE_TAG` | Ignore tag folder when checking local binary path                                                            |
| `PKG_NODE_PATH`  | Custom path to the local nodejs binary to use                                                                |
| `HTTPS_PROXY`    | Optional HTTPS proxy to use when fetching binaries                                                           |
| `HTTP_PROXY`     | Optional HTTP proxy to use when fetching binaries                                                            |
| `MAKE_JOB_COUNT` | Number of parallel jobs when building binaries (value passed to `make -j` option). Default to number of cpus |
| `CFLAGS`         | Flags to use when invoking C compiler                                                                        |
| `CXXFLAGS`       | Flags to use when invoking C++ compiler                                                                      |
| `STRIP`          | Path to `strip` command. Default to `strip`                                                                  |
