
# Macro

- [Macro](#macro)
  - [Known issues](#known-issues)
  - [AST](#ast)
  - [Types of Macros](#types-of-macros)

[Use macros to generate code at compile time](https://developer.apple.com/documentation/swift/applying-macros/).
Macros transform your source code when you compile it, letting you avoid
writing repetitive code by hand. During compilation, Swift expands any
macros in your code before building your code as usual.

## Known issues

> [!NOTE]
> There's a few issues with employing Macros into a project that our outlined
> by [Stephen Celis](https://forums.swift.org/t/macro-adoption-concerns-around-swiftsyntax/66588):
>
> 1. Introducing SwiftSyntax to a project immediately incurs an additional 20
> second debug build cost to a project.
> 2. Because SwiftSyntax is a moving target and versioned alongside Swift
> releases, how can a library adopt macros and be compatible with multiple
> Swift versions at the same time?
> 3. SwiftSyntax’s API seems to be in constant flux. I’ve used the library
> sporadically over the years and whenever I return to a project that uses
> it, it rarely still builds.

Since then, a few PRs have been merged to fix this issue. The latest one adds
a prebuilt swift-syntax to speed up the build: [Swift-Syntax Prebuilts for Macros
6.1](https://github.com/swiftlang/swift-package-manager/pull/8214).

## AST

[Swift AST Explorer](https://swift-ast-explorer.com) is a good website to visualize
Swift AST and select nodes within the editor, a great way to learn about the
structure of Swift syntax trees.

## [Types of Macros](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/macros/)

There's two types:

- Freestanding
- Attached
