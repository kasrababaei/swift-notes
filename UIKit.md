# UIKit

- [UIKit](#uikit)
  - [Why `required`?](#why-required)

## Why `required`?

When creating a subclass of `UIView`, if you want the view to work
with Interface Builder (storyboards or XIB files), you must implement:

```swift
required init?(coder aDecoder: NSCoder)
```

This is because UIKit will use this initializer to unarchive the view from a
storyboard or XIB file.
