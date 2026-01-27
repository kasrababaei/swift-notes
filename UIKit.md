# UIKit

- [UIKit](#uikit)
  - [Why `required`?](#why-required)
  - [awakeFromNib](#awakefromnib)
  - [Storyboards vs Programmatically UI](#storyboards-vs-programmatically-ui)

## Why `required`?

When creating a subclass of `UIView`, if you want the view to work
with Interface Builder (storyboards or XIB files), you must implement:

```swift
required init?(coder aDecoder: NSCoder)
```

This is because UIKit will use this initializer to unarchive the view from a
storyboard or XIB file.

## awakeFromNib

This method is deprecated in visionOS 1.0.

## Storyboards vs Programmatically UI

Believe it or not, even in 2026, there's still people who strongly believe
creating UIs using Storyboards is better than doing it programmatically.

Here's a list of disadvantages of using Storyboards:

- Each Storyboard has an associated Swift file. This creates two sources of
  truth. When setting up the view, e.g., setting the `backgroundColor` or
  the constraints, it's unclear whether it should be done in the code or
  the Storyboard. Any animation or dynamic changes need to be done in the
  code.
- Refactoring or modifying a complex Storyboard can be time consuming.
- Storyboards are essentially `XML` files. Reading and reviewing `XML` files
  isn't an easy tasks during code review. In most cases, developers need to
  pull the branch and use Xcode to open the file. Also, the `XML` files usually
  contain hundreds of lines compared to a few lines of Swift codes that could
  deliver the same. Frameworks like [SnapKit](https://github.com/SnapKit/SnapKit)
  have made it using Auto Layout very easy and convenient.
- Related to the previous point, Storyboards cause merge conflicts very often.
- Finding references to a specific view on a Storyboard isn't easy.
- Refactoring the project and moving Storyboards can break things that only
  surface themselves at the runtime. For instance, trying to instantiate a
  nib file that's moved from one module to another module only fails at runtime.
- Applying design system updates, such as changing font size or different spacing,
  requires manual work.
- Looking how SwiftUI views can only be built programmatically, using Storyboards
  is a misalignment that introduces a dissonance and makes UIKit and SwiftUI integration
  very challenging.
