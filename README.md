# `asimov-platform/rust-build-action`

Highly customizable and easy to use action that builds and uploads Rust crates for any target.

Most of the times you would want to use this action in combination with [asimov-platform/release-action](https://github.com/asimov-platform/release-action), or something similar.

## Features

- Highly customizable (working directory, checkout, upload, etc).
- Support for Windows, Linux, and macOS runners.
- Support for cross-compilation.
- Support for [cargo-zigbuild](https://github.com/rust-cross/cargo-zigbuild).
- Rust toolchain selection (channel, version, etc).
- Installing MacOS SDK if needed.
- Automatically installs [MinGW](https://www.mingw-w64.org/) if needed.
- Stripping & compressing.
- And more (see [Inputs](#inputs))!

> [!IMPORTANT]
> Please note that the strategy I use to select correct package to build is very naive and
> probably will not work if you have a workspace. In that case please use
> the `package-name` input to specify it instead.

## Tested runners and targets

### Ubuntu 24.04

| Target                      | Working | Zigbuild |
| --------------------------- | ------- | -------- |
| `x86_64-unknown-linux-gnu`  | ✅      | -        |
| `aarch64-unknown-linux-gnu` | ✅      | -        |
| `x86_64-apple-darwin`       | ✅      | ✅       |
| `aarch64-apple-darwin`      | ✅      | ✅       |
| `x86_64-pc-windows-gnu`     | ✅      | -        |
| `x86_64-pc-windows-msvc`    | ❌      | -        |

### macOS 14.7

| Target                 | Working | Zigbuild |
| ---------------------- | ------- | -------- |
| `x86_64-apple-darwin`  | ✅      | -        |
| `aarch64-apple-darwin` | ✅      | -        |

### Windows 2022

| Target                    | Working | Zigbuild |
| ------------------------- | ------- | -------- |
| `x86_64-pc-windows-gnu`   | ✅      | -        |
| `x86_64-pc-windows-msvc`  | ✅      | -        |
| `aarch64-pc-windows-msvc` | ✅      | -        |

## Usage

### Inputs

```yaml
- uses: asimov-platform/rust-build-action@v5
  with:
    # Target to build for.
    # Required.
    target:

    # Profile to build with.
    # Optional. Defaults to release.
    profile:

    # Version of GLIBC to link to.
    # Only works on Linux and using zigbuild.
    # Optional. Defaults to system default.
    glibc-version:

    # Prefix for the output artifact name.
    # Optional. Defaults to empty string.
    artifact-prefix:

    # Suffix for the output artifact name.
    # Optional. Defaults to target.
    artifact-suffix:

    # Name of the output artifact. Must be unique.
    # Prepended with artifact-prefix and appended with artifact-suffix.
    # Optional. Defaults to the binary being built.
    artifact-name:

    # Name of the packages to build, separated by space.
    # Optional. Defaults to workspace's default-members that have at least one binary target.
    packages-to-build:

    # Name of the targets to build.
    # Optional. Defaults to all binary targets in the packages-to-build.
    targets-to-build:

    # Extension for the output binary file.
    # Optional. Defaults to empty string.
    binary-extension:

    # Working directory.
    # Optional. Defaults to root.
    working-directory:

    # Checkout repository?
    # Optional. Defaults to true.
    checkout:

    # Rust toolchain to use.
    # Optional. Defaults to stable.
    rust-toolchain:

    # Install MacOS SDK?
    # Optional. Defaults to false.
    install-macos-sdk:

    # Use cargo-zigbuild?
    # Optional. Defaults to false.
    use-zigbuild:

    # Extra arguments to pass to cargo build.
    # Optional. Defaults to empty string.
    extra-build-arguments:

    # Strip symbols?
    # Optional. Defaults to false.
    strip-artifact:

    # Compress artifact?
    # Optional. Defaults to true.
    compress-artifact:

    # Upload artifact?
    # Optional. Defaults to true.
    upload-artifact:
```

### Outputs

| Name           | Description                                                                                                                                                                                                                                                                                   | Example                                                                     |
| -------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `target-dir`   | Path to the build directory (target directory).                                                                                                                                                                                                                                               | `target/x86_64-unknown-linux-gnu/release`                                   |
| `output-dir`   | Path to the output directory (relative to the working directory).                                                                                                                                                                                                                             | `output`                                                                    |
| `output-paths` | Paths to the output files separated by space (relative to the working directory).                                                                                                                                                                                                             | `output/my-crate-windows-x64.zip output/my-crate-linux-gnu.tar.gz`          |
| `artifact-id`  | GitHub ID of an Artifact, can be used by the REST API.                                                                                                                                                                                                                                        | `1234`                                                                      |
| `artifact-url` | URL to download an Artifact. Can be used in many scenarios such as linking to artifacts in issues or pull requests. Users must be logged-in in order for this URL to work. This URL is valid as long as the artifact has not expired or the artifact, run or repository have not been deleted | `https://github.com/example-org/example-repo/actions/runs/1/artifacts/1234` |

## Examples

### Build, strip, compress and upload a Linux binary using latest stable Rust toolchain

```yaml
steps:
  - uses: asimov-platform/build-rust-action@v5
    with:
      target: x86_64-unknown-linux-gnu
      artifact-name: linux-x64
      strip-artifact: true
      compress-artifact: true
      rust-toolchain: stable
```

### Build, compress and upload a macOS binary using Rust version 1.81.0

```yaml
steps:
  - uses: asimov-platform/build-rust-action@v5
    with:
      target: aarch64-apple-darwin
      artifact-name: macos-arm64
      compress-artifact: true
      rust-toolchain: 1.81.0
```

### Build, compress and upload a Windows binary with specific features using nightly Rust toolchain

```yaml
steps:
  - uses: asimov-platform/build-rust-action@v5
    with:
      target: x86_64-pc-windows-gnu
      artifact-name: windows-x64
      binary-extension: .exe
      compress-artifact: true
      rust-toolchain: nightly
      extra-build-arguments: --features foo,bar
```
