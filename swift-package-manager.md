# Swift Package Manager

- [Swift Package Manager](#swift-package-manager)
  - [Conditionally Include Development Dependencies](#conditionally-include-development-dependencies)
  - [DEBUG/DEVELOPMENT Configurations Name for SPM](#debugdevelopment-configurations-name-for-spm)
  - [Checksum](#checksum)
  - [SPM vs XCFramework](#spm-vs-xcframework)

## Conditionally Include Development Dependencies

In SPM, it's possible to add a condition to exclude/include packages based
on some conditions. Following [this issue on swift-snapshot-testing](https://github.com/pointfreeco/swift-snapshot-testing/issues/201),
can pass environment variables through terminal and then read them inside
`Package.swift` like [this commit](https://github.com/pointfreeco/swift-tagged/commit/77bdcc17f31040bc1b415a1bbd457e400b8aa385):

```bash
DEVELOP=1 swift run xcodegen
```

It's also possible to use preprocessor flags like [this commit](https://github.com/Quick/Quick/blob/a0e1457029c2a451a321fdd3a6a6f36ac367010f/Package.swift)
from Quick.

## DEBUG/DEVELOPMENT Configurations Name for SPM

Currently, when using Swift Package Manager packages in Xcode, SPM compiles
the package with reference to the name of the Build Configuration and
automatically selects whether to compile with debug or release, which
determines the compilation flag like DEBUG and this determines the architecture
of the final binary. This automatic selection may cause problems when using
a custom Build Configuration in Xcode other than the default “Debug” and “Release”.

Right now (October 2022) there is no particularly good way to map the Build
Configuration in Xcode to the SPM build environment <sup>[*](https://www.sobyte.net/post/2022-10/spm-in-xcode/#determination-based-on-build-configuration)</sup>.

## Checksum

Remote binary targets require you to provide a checksum to verify that the
hosted archive file matches the archive you declare in the manifest file.
Not all packages provide such checksum. It's possible to get the checksum
from a ZIP file that contains a `Package.swift`:

```bash
shasum -a 256 SwiftLintBinary-macos.artifactbundle.zip | sed 's/ .*//'                 
Prints: 227258fdb2f920f8ce90d4f08d019e1b0db5a4ad2090afa012fd7c2c91716df3
```

_[Source](https://www.avanderlee.com/swift/binary-targets-swift-package-manager/)_

## SPM vs XCFramework

IDE / Xcode integration:

- Package indexing on every Xcode launch, even with no changes — blocks code
  completion and jump-to-definition until it finishes
- SourceKit indexes all SPM source files, increasing total index size and slowing
  symbol lookup across the whole workspace
- Adding/removing packages modifies `project.pbxproj` in non-obvious ways, making
  diffs noisy and increasing merge conflict surface

Reliability:

- SPM cache in `~/Library/Caches/org.swift.swiftpm/` and DerivedData can become
 corrupted — fix is manual deletion, which triggers a full re-clone
- "Package resolution" step in CI needs network access; flaky or rate-limited
  GitHub/registry connections cause intermittent failures
- Background package validation in Xcode can silently trigger re-resolution mid-session

Dependency management:

- Package.resolved merge conflicts are frequent on active teams
- Diamond dependency problem: if two packages require conflicting versions of
  a third, resolution fails with cryptic errors
- Transitive dependencies are exposed to your build — you inherit their Swift
  version requirements and platform constraints even for things you don't use directly

Build system:

- SPM has its own build system that sits somewhat awkwardly inside Xcode's;
  custom build phases, xcconfig overrides, and code signing sometimes conflict
  with SPM targets
- No clean way to have different packages per build configuration (Debug vs. Release)
- xcodebuild can trigger full package resolution on invocations where you'd
 expect incremental behavior

XCFrameworks sidestep almost all of these because from Xcode's perspective
they're just files on disk — no source to index, no resolution to run,
no network needed.
