# Swift on Linux: toolchain research

Status: **not wired in.** `ipawiz` emits Swift-ready Theos Makefiles and
warns clearly when a Swift compiler is missing (`_swift_preflight`), but no
code installs or configures a Swift-capable toolchain yet. This document
records what it would take, so the follow-up isn't re-researched from zero.

## The problem

`ipawiz --setup` installs sbingner's `linux-ios-arm64e-clang-toolchain`
(clang 10, 2020). It contains **no Swift compiler** (`swiftc`/`swift`).
So any `.swift` source fails at `internal-<App>-swift-support` with a
Theos `Error 2`, even though the Objective-C/C parts of the same app
build fine.

This is two independent problems, often conflated:

1. **A better clang/Obj-C toolchain.** sbingner's clang 10 is old, links
   against the dropped `libtinfo.so.5`, and can't compile against modern
   iOS SDK headers (hence the `--web`/`--app` SDK pin to 14.5).
2. **Swift.** A separate compiler + Swift iOS SDK + matching runtime.

Fixing (1) does **not** fix (2).

## L1ghtmann's toolchain does not solve Swift

The toolchain follow-up flagged in the `--setup` PR was L1ghtmann's
`Building-An-Ios-Toolchain`. Per its own docs it builds **only**
`llvm`, `clang`, `ldid`, `libtapi`, and `cctools-port` — C/C++/Objective-C.
There is **no `swiftc`**. It would be a good answer to problem (1)
(newer clang, no libtinfo5, modern SDKs ⇒ could drop the SDK 14.5 pin),
but it does nothing for Swift.

## The actual Swift-on-Linux path

Theos *does* support Swift on macOS, iOS, and Linux. On Linux the
supported route is the **official open-source Swift toolchain from
swift.org** (the Linux build), used to cross-compile to iOS. Theos's
`swift-support` submodule (already cloned by `--setup --recursive`)
drives this when a Swift compiler is discoverable.

Key facts established by the research:

- Use the swift.org Linux toolchain; put its `swiftc`/`swift` on `PATH`
  (or inside `$THEOS/toolchain/linux/iphone/bin`) so Theos finds it.
- Cross-compiling Swift to iOS `arm64` from a Linux Swift toolchain is
  finicky and version-sensitive: the Swift release, the bundled clang,
  and the iOS SDK must be mutually compatible. Theos describes its Linux
  Swift support as "preliminary."
- On-device runtime: the Swift runtime ships with iOS as of **iOS 12.2**.
  Apps `ipawiz` generates target iOS 14.0, so no extra runtime package is
  needed on a supported device.

## Recommended next steps (the follow-up)

1. Add an opt-in `--setup --swift` that downloads a pinned swift.org
   Linux toolchain and exposes `swiftc` where Theos looks for it.
2. Pin a known-good triple of {Swift release, clang, iOS SDK} and test a
   trivial SwiftUI/`@main` app end-to-end to a valid `arm64` `.app`.
3. Test across distros (Ubuntu 22.04/24.04, Fedora, Arch) — the swift.org
   Linux builds are per-distro and dependency-sensitive.
4. Only then flip `_swift_preflight` from "warn" to a supported path, and
   consider replacing sbingner's clang 10 with a newer toolchain
   (L1ghtmann's or similar) to also retire the SDK 14.5 pin.

Until that lands, `ipawiz`'s behavior is deliberate: generate correct
Swift-aware Makefiles, build the non-Swift parts, and tell the user
exactly why Swift didn't compile and where the work is tracked.

## Sources

- [theos/theos — Swift wiki](https://github.com/theos/theos/wiki/Swift)
- [Swift · Theos docs](https://theos.dev/docs/swift)
- [theos/theos discussion #615 — building a Swift iOS toolchain](https://github.com/theos/theos/discussions/615)
- [theos/theos Wiki — Installation (Linux)](https://github.com/theos/theos/wiki/Installation-Linux)
- [L1ghtmann/Building-An-Ios-Toolchain](https://github.com/L1ghtmann/Building-An-Ios-Toolchain)
- [swiftlang/llvm-project](https://github.com/swiftlang/llvm-project)
