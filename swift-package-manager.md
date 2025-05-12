# Swift Package Manager

- [Swift Package Manager](#swift-package-manager)
  - [Conditionally Include Development Dependencies](#conditionally-include-development-dependencies)
  - [DEBUG/DEVELOPMENT Configurations Name for SPM](#debugdevelopment-configurations-name-for-spm)
  - [Checksum](#checksum)

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
