# Swift Package Manager

- [Swift Package Manager](#swift-package-manager)
  - [Conditionally Include Development Dependencies](#conditionally-include-development-dependencies)

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
