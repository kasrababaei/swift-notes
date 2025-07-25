# SwiftUI

> [!NOTE]
> Views are functions of state, not of a sequence of events.

- [SwiftUI](#swiftui)
  - [General Notes](#general-notes)
  - [View Builders](#view-builders)
  - [Group](#group)
  - [Dynamic Views vs Static Views](#dynamic-views-vs-static-views)
  - [Lifetime](#lifetime)
  - [Identity](#identity)
  - [State](#state)
  - [StateObject](#stateobject)
  - [ObservedObject](#observedobject)
  - [Binding](#binding)
    - [Binding vs Bindable](#binding-vs-bindable)
  - [View Updates and Performance](#view-updates-and-performance)
    - [Which property wrapper to use](#which-property-wrapper-to-use)
  - [Layout](#layout)
    - [Leaf Views](#leaf-views)
      - [Text](#text)
      - [Shapes](#shapes)
      - [Colors](#colors)
      - [Image](#image)
      - [Divider](#divider)
      - [Spacer](#spacer)
    - [View Modifiers](#view-modifiers)
      - [Padding](#padding)
      - [Fixed Frames](#fixed-frames)
      - [Flexible Frames](#flexible-frames)
      - [Aspect Ratio](#aspect-ratio)
      - [Overlay and Background](#overlay-and-background)
      - [Fixed Size](#fixed-size)
    - [Container Views](#container-views)
      - [HStack and VStack](#hstack-and-vstack)
      - [ZStack](#zstack)
      - [Scroll View](#scroll-view)
      - [Geometry Reader](#geometry-reader)
      - [List](#list)
      - [LazyHStack and LazyVStack](#lazyhstack-and-lazyvstack)
      - [LazyVGrid and LazyHGrid](#lazyvgrid-and-lazyhgrid)
      - [ViewThatFits](#viewthatfits)
    - [Rendering Modifiers](#rendering-modifiers)
      - [Alignment](#alignment)
      - [Modifying Alignment Guides](#modifying-alignment-guides)
      - [Custom Alignment Identifiers](#custom-alignment-identifiers)

## General Notes

Sometimes the entity which drives the view should have limit access to its
properties. If done naively, one can observe far too much state than is necessary
and cause slow, under-performing views. It is usually best to whittle down state
to the bare essentials the view needs, especially when you have lots of features
composed together.

In cases where we need to force the view to be rendered, for instance in testing,
can use a private API called `_render(seconds:)` but it's only available on
`UIHostingController`; so, need to wrap the view in that.

## View Builders

A SwiftUI syntax for constructing list of views, which is built on top off Swift's
result builder.

## Group

Group is a layout-agnostic abstraction around a view builder.
When placing a `Group`, including the modifiers, as the root view or as the only
subview within a scroll view, the group behaves like a `VStack` and the modifiers
aren't applied to each individual view within the group. When placing a `Group`
within an `Overlay` or `Background`, it behaves like an implicit `ZStack`.

## Dynamic Views vs Static Views

```Swift
HStack {
    Text("Hello") // Static view
    if showText {
        Text(", World!") // dynamic view
    }
}

// View Tree:

        HStack
          |
          |
    -------------
    |           |
    |           |
   Text       Text?
```

SwiftUI has something called the `attribute graph`, which includes more than just
the rendered views; it also contains the state and tracks dependencies. Apple
calls the nodes in the render tree `attributes`.

`Nodes` in a render tree have a lifetime which is not bound to whether the view
is onscreen or not. For example, a `LazyVStack` renders lazily as opposed to a
`VStack` that does it _eagerly_, still, the nodes in the render tree are preserved
when they go offscreen. Bottom line is, we have no control over the lifetime.

## Lifetime

SwiftUI provides three hooks into lifetime events:

- `onAppear`: can be called multiple times
- `onDisappear`: the counter part of an `onAppear`
- `task`: combination of the two used for asynchronous work. This modifier creates
a new `Task` at the point where `onAppear` would be called, and it cancels this
`Task` when `onDisappear` would be invoked.

## Identity

_Implicit identity_ is the position in the view tree which is used by
SwiftUI Since it doesn't have reference types.

```Swift
HStack {
    Image(systemName: "hand.wave")
    if let g = greeting {
        Text(g)
    } else {
        Text("Hello")
    }
}

// This is just a human-friendly model to demonstrate

        HStack
          |
          |
    -----------------
    |               |
    |               |
    0               |
    |               |
  Image     Conditional content
                    |
              ---------------
              |             |
           if branch   else branch
              |             |
            Text           Text
```

Views can also have an _explicit identity_. This is mostly for views in a
`ForEach` where each item is assigned an explicit identifier. For example,
a unique identifier of the underlying data (either by conforming the items to the
`Identifiable` protocol or by providing a `KeyPath` to a unique identifier).
However, we can also assign explicit identifiers using the id modifier.

```Swift
var body: some View {
    Text("Hello").id("")
}
```

The id parameter can be any `Hashable` value.

Try not using patterns that create conditional content/branch in the view tree
that might have unforeseen consequences.

> It is essential to keep your view hierarchy without unnecessary branches that
> you may create using if statements in the body of a `ViewBuilder` closure
> because it may hurt the performance of your views and produce state losses.
> _[From Swift with Majid](https://swiftwithmajid.com/2023/05/03/the-power-of-overlays-in-swiftui/)_.

Also, with reference to [Demystify SwiftUI](https://developer.apple.com/videos/play/wwdc2021/10022/?time=2215)
from WWDC23, branches are a form of structured identity. This means we have two
copies of the content instead of a single, optionally modified one. By removing
the branch, we can describe the view as having just one identity. By moving the
branching condition into a modifier, can help performance because the dependant
code becomes tightly scoped. So, when the condition changes, only the modifier
needs to change. This modifiers are called _inert modifiers_ because they don't
affect the rendered result. Because there is no resulting visual effect, the
framework can efficiently prune away the modifier, furthermore reducing the cost.

In short, as mentioned in [the video](https://developer.apple.com/videos/play/wwdc2021/10022?time=648),
an if statement defines different views for each conditional branch. This
will cause the views to transition in and out because SwiftUI understands that
each branch of the if statement represents a different view with a distinct
identity. Alternatively, we could just have a single PawView that changes its
layout and color. When it transitions to a different state, the view will
smoothly slide to its next position. That's because we're modifying a single
view with a consistent identity. Both of these strategies can work, but
SwiftUI generally recommends the second approach. By default, try to preserve
identity and provide more fluid transitions. This also helps preserve your
view's lifetime and state.

_Future reading list:_

- [Conditionally apply modifier in SwiftUI](https://forums.swift.org/t/conditionally-apply-modifier-in-swiftui/32815/29)
- [WWDC23 Demystify SwiftUI Performance](https://developer.apple.com/videos/play/wwdc2023/10160)
- [Why Conditional View Modifiers are a Bad Idea](https://www.objc.io/blog/2021/08/24/conditional-view-modifiers/)

Instead, use the following pattern:

```Swift
HStack { 
    Image(systemName: "hand.wave")
    Text("Hello")
      .background(highlights ? .red : .clear)
}

// View tree
HStack
          |
          |
    -----------------
    |               |
    |               |
  Image        .background
                    |
              ---------------
              |             |
              |             |
            Text           Color
```

## State

As of iOS 17, SwiftUI doesn't depend on `Combine` and instead uses macro-based
solution. `@State` property wrapper is now used for values and objects, whereas
we usually only used it for values pre-iOS 17.

`State` is meant to be used for private view state values. During the execution
of the body property, SwiftUI notices the state property is accessed; so, it adds
a dependency between the state property and the view's node in the render tree.

If the state property is not re-rendered inside the `body`, SwiftUI won't rerender
the `body` when the state property changes.

Here's how we could write the same code without using the `@State` property wrapper:

```Swift
struct Counter: View {
    private var _value: State(initialValue: 0)
    private var value: Int {
        get { _value.wrappedValue }
        nonmutating set { _value.wrappedValue = newValue }
    }

    var body: some View {
        Button("Increment: \(value))") {
            value += 1
        }
    }
}
```

The `State(initialValue:)` initializer makes it clear that the value 0 is just the
_initial_ value of the state property. This is the value that will be used when the
node for the counter view is first created in the render tree. Once the node is
there, the initial value of the state property will be ignored.

## StateObject

The `@StateObject` property wrapper works much in the same way as `@State`: we
specify an initial value (an object in this case), which will be used as the
starting point when the node in the render tree is created. From then on, SwiftUI
will keep this object around across renders for the lifetime of the node in the
render tree. It  also observes the object for changes via the `ObservableObject`
protocol.

Observed objects marked with the `@StateObject` property wrapper don’t get
destroyed and re-instantiated at times their containing view struct redraws<sup>[*](https://www.avanderlee.com/swiftui/stateobject-observedobject-differences/)</sup>.

`Observable` framework in iOS 17 replaced the entire existing object-observation
model based on the `Combine` framework. The `Observable` macro does two things:

1. adds conformance to the `Observable` marker protocol
2. changes object's properties to track both read and write access.

The macro only forms a dependency between the view and access properties. This
is more efficient. It shifts observation from the object level to the individual
properties of the object.

The new `@Observation` macro is more efficient than using the `@ObservableObject`
property wrapper. If you only use one property of an object in your view's body,
changes to the other properties won't cause redraws of this view. This can reduce
unnecessary view updates and thus improve performance. Also, observable objects
can be nested in optionals, arrays, or other collections. Observation and view
updates will continue to work as expected, because of the property-level tracking
of dependencies in the view's body.

Two common _mistakes_ with regard to observing objects:

1. to use `@State` when the lifetime of an object is managed somewhere outside
of the view.
2. not using `@State` when an object's lifetime isn't managed somewhere outside
of the view.

In the example below, the problem is that at the time the initializer runs, the
view doesn't yet have _identity_. So, all this initializer does is change the
initial value of the `@State` property, but doesn't affect the state that's
being used when the view is already onscreen.

```Swift
struct Counter: View {
    @State var model: Model

    init(model: Model) {
        self.model = model
    }

    var body: some View {
        Button("\(model.value)") { model.value += 1 }
    }
}
```

`@Published` property wrapper is syntactic sugar for

```Swift
final class Counter: ObservableObject {
    var value: Int {
        willSet { objectWillChange.send() }
    }
}
```

If we implement our own `objectWillChange` publisher for some reason, `@Published`
doesn't work.

The `Counter` view is created, but value of the state isn't present yet, because
the initial value is stored as an `autoclosure`:

```Swift
struct CounterView: View {
    @StateObject private var model = Model()

    var body: some View {
        Button("Increment: \(model.value))") {
            model.value += 1
        }
    }
}
```

## ObservedObject

The `@observedObject` property wrapper is much simpler than `StateObject`:

- it doesn't have the concept of an initial value
- it doesn't maintain the observed object across renders

All it does is subscribe to the object's `objectWillChance` publisher and rerender
the view when this publisher emits an event. This makes `ObservedObject` the only
correct tool if we want to explicitly pass objects from the outside into a view
(when targeting platforms before iOS 17). This is the equivalent of an Observable
object within a regular property.

## Binding

When writing components, it's often the case that we need read and write access
to a value, but don't need to know what the source of truth for this value actually
is. Bindings are used exactly for this purpose, as we can see with many of SwiftUI's
built-in components.

The code below, shows how the code would look like without the binding

```Swift
struct Counter: View {
    var value: Int
    var setValue: (Int) -> ()
    var body: some View {
        Button("Increment: \(value)" { setValue(value + 1) })
    }
}

// Using `Counter`
struct CounterView: some View {
    @State private var value = 0
    var body: some View {
        Counter(value: value, setValue: { value = $0 })
    }
}

// Or this version
struct Counter: View {
    var _value: Binding<Int>
    var value: Int {
        get { _value.wrappedValue }
        set { _value.wrappedValue = newValue }
    }
    init(value: Binding<Int>) {
        self._value = value
    }
    var body: some View {
        Button("Increment: \(value)") { value +=1 }
    }
}

// The initialize's only job is to expose a nicer API without an
// underscored `_value` label for its parameter:
struct ContentView: View {
    @State private var value = 0
    var body: some View {
        Counter(value: Binding(get: { value }, set: { value = $0 }))
    }
}
```

To demystify the code above, we often use the syntactic sugar below:

```Swift
struct Counter: View {
    @Binding var value: Int
    var body: some View {
        Button("Increment: \(value)" { value += 1 })
    }
}

struct CounterView: View {
    @State var value: Int= 0
    var body: some View {
        Counter(value: $value)
    }
}
```

The `$value` is shorthand for `_value.projectedValue`. The dollar syntax isn't
specific to SwiftUI; rather, it's a feature of Swift's [property wrappers](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/properties/#Property-Wrappers).

### Binding vs Bindable

Since we often no longer need to use a property wrapper when working with
the `@Observable` macro, we can't construct a binding in the same way as when
using, for example, the `@ObservedObject` property wrapper. Instead,
the `Bindable` wrapper, which you can use either as a property wrapper or
directly inline, was introduced:

```Swift
struct Counter: View {
    @Bindable var model: Model
    var body: some View {
        Stepper("Increment: \(model.value)", value: $model.value)
    }
}

struct CounterView: View {
    var model = Model.shared
    var body: some View {
        Counter(model: model)
    }
}
```

## View Updates and Performance

If we pull our state in one large observable object (not using the new
`@Observable` macro), SwiftUI has to rerender all views that observe that
object on any change, even if the particular change perhaps only affected a
small subset of the views that observe the object. The `@Observable` macro
alleviates this problem to a large degree, because it shifts observation
from the object level to the individual properties of the object.

When running into performance issues, there are several techniques to diagnose
which view bodies are being executed. The first option is to insert a print
statement in view's body:

```Swift
var body: some View {
  let _ = print("Executing <MyView> body")
   Text("Hello, World!")
}
```

The _anonymous variable assignment_ `let _ = ...` is necessary because the view
builder only accepts expressions of type `View`. We cannot simply call `print`
because the return type of print function is `Void`., which the view builder
cannot handle.

To find out _why_ a view's body was re-executed, we can use the `Self._printChanges()`
API in the view body like this:

```Swift
var body: some View {
  let _ = Self._printChanges()
  Text("Hello, World!")
}
```

This print statement will log the reason of the rerender to the console:

- If the view was rerendered due to a state change -> Name of the state property
will be logged in its underscored form
- If the view value itself has changed, i.e., if the value of one of the view's
properties has changed -> `@Self` will be logged
- If the identity of the view has changed -> `@identity` will be logged. I
practice, this usually means the view was freshly inserted into the rendered tree.

### Which property wrapper to use

- Start with a regular property without any property wrappers
- When a view needs to read and write access to a value (not an object) and it
should own that value as local, private view state, then we use `@State`
- If the view needs read and write access to a value but shouldn't own that
value (and doesn't care where the value is actually stored), then we use `@Binding`
- If a view needs state in the form of an object and the view should own that
object as local, private view state, which cannot be passed in from the outside,
then we use `@StateObject` pre-iOS 18 or `State` with an `Observable` object
post-iOS 17
- If we need an object to be passed in from the outside, then we use
`@ObservableObject` pre-iOS 17 and just a plain property post-iOS 17

A good rule of thumb is that `State` and `StateObject` should only be used if a
property can be initialized directly on the line where it's declared.If that
doesn't work, we should probably use `@Binding`, `@ObservedObject`, or a plain
property with an `@Observable` object.

## Layout

The first thing to keep in mind is that SwiftUI's layout algorithm proceeds
top-down along the view tree.

```Swift
VStack {
  Image(systemName: "globe")
  Text("Hello, World!")
}
```

Since the `VStack` is the root view in the above example, it'll receive the safe
screen area as the proposed size. To determine its own size, the stack first
recursively proposes sizes to its subviews. The image will report its size based
on the size of the globe symbol, and the text will report its size based on the
proposed size and the string it has to render. The stack computes its own size
as the size of the union of its subviews' frames and reports that back to the
window. The size that the subview reports to its parent is the definitive size
and the parent cannot alter this size unilaterally.

In iOS 16, there's a type in the `Layout` protocol called `ProposedViewSize`:

```Swift
struct ProposedViewSize {
    var width, height: CGFloat?
}
```

The difference between `ProposedViewSize` and `CGSize` is that both components
are optional in `ProposedViewSize`. Proposing `nil` for a component means that
the view can become its _ideal size_ in that dimension. The ideal size is
different for each view.

In API terms, we can express the layout algorithm like this:

1. The parent calls `sizeThatFits` on its subview, passing on the proposed size.
2. The subview determines its own size based on the proposed size, perhaps
calling `sizeThatFits` on its subviews if it has any. Some views completely
disregard the proposed size (e.g. `Image` does so by default). When a view
returns the proposed size, we say that it _accepts_ the proposed size.
3. The subview reports its own size to its parent via the return value of
the `sizeThatFits` method.
4. The parent places the subview according to its own alignment and the alignment
guides of the subview.

### Leaf Views

Views that have no subviews. Examples:

1. `Text`
2. `Shape`
3. `Color`
4. `Divider`
5. `Spacer`

#### Text

By default, `Text` views fit themselves into any proposed size. `Text` uses
various strategies to make that work, in this order:

- it'll break the text into multiple lines (word wrapping)
- break up words (line wrapping)
- truncate
- clip the text

`Text` always reports back the exact size it needs to render the content, which
is less than or equal to the proposed width, and at least the height of one line
(with the exception of proposing 0⨉0).

`lineLimit(_ number:)` modifier lets us specify the maximum number of lines that
should be rendered. Specifying `nil` means no line limit.

`lineLimit(_:reservesSpace:)` modifier lets us specify the maximum number of
lines that should be rendered, while giving us the option to always include the
space for these lines in the reported size, regardless of whether or not they're
empty.

If we apply `.fixedSize()` to `Text`, it'll become its ideal size, because
`fixedSize` proposed `nil ⨉ nil` of the text. The ideal size of the text is the
size that's needed to render the content without wrapping and truncation.

Assuming there's a window with a safe area of 320 ⨉ 480:

```Swift
Text("Favorite")
  .padding(10)
  .background(Color.teal)
```

in API terms, the layout algorithm can be expressed like:

```Swift
    View Tree
        1|8
         |
    background
  ---------------
  |             |
2 | 5          6|7
  |             |
padding       Color
  |
3 | 4
  |
 Text
```

1. The system proposes 320⨉480
2. The background proposes the same size to its primary subview (the padding)
3. The padding subtracts 10 points on each edge, and it proposed 300⨉460 to the text
4. The text reports its size as 51⨉17
5. The padding adds 10 points on each edge, and it reports its size as 71⨉37
6. The background proposes the size of the padded text (71⨉73) to the secondary
subview (color)
7. The color accepts and reports the proposed 71⨉73
8. The background reports the size of its primary subview (71⨉73)

For debugging a view:

1. Generally adding a border to the view is sufficient.
2. Alternatively, can add an overlay with a geometry reader to also render the
size of the view.

It's also to add the following statement to find out what caused a re-render:

```swift
struct ContentView: View {
 @StateObject private var evilObject = EvilStateObject()

 var body: some View {
  let _ = Self._printChanges()
  Text("What could possibly go wrong?")
 }
}
```

#### Shapes

Most built-in shapes (`Rectangle`, `RoundedRectangle`, `Capsule`, and `Ellipse`)
accept any proposed size from zero to infinity and fill the available space.
`Circle` is an exception: it'll fit itself into any proposed size and report
back the actual size of the circle. If we propose `nil` to a shape, i.e., wrap
it in a `.fixedSize`, it takes on a default size of 10⨉10.

#### Colors

When using a color directly as a view, e.g., `Color.red`, it behaves just
as `Rectangle().fill(...)` from a layout point of view.

_If we put a color in a background that touches the non-safe area, color will
magically bleed into the non-safe area. If we want to prevent this, we can use
the `ignoreSafeAreaEdges` on `.background`, or use `Rectangle().fill(...)`
instead of `Color`._

`fill(.red)` vs `background(.red)`: _`fill` is for filling a shape such as a
circle while `background` will set the background color of the whole view,
i.e., the container of the circle._

#### Image

By default, `Image` views report a static value: the size of the underlying image.
Once we call `.resizable()` on an image, it makes the view completely flexible:
the `Image` will then accept any proposed size, report it back, and squeeze the
image into that size.

In practice, virtually any resizable image will be combined with an
`.aspectRatio(contentMode:)` or `.scaleToFit()` modifier to prevent the image
from being distorted.

#### Divider

When `Divider` is used outside of horizontal stacks, it accepts any proposed
width and reports the height of the divider line. Within a horizontal stack,
the divider accepts the proposed height and reports the width of the divider line.

Proposing `nil` will result in a default size of 10 on the flexible axis,
depending on the context.

#### Spacer

Outside of horizontal or vertical stacks, `Spacer` accepts any proposed size,
from its minimum length to infinity. However, within a vertical stack, `Spacer`
accepts height from its minimum length to infinity, but it reports a width of zero.
Within a horizontal stack, it behaves the same way (wth the axes swapped).
The minimum length of a spacer is the length of the default padding, unless
the length is specified using the `minLength` parameter in the spacer's initializer.

A popular use of spacer is for aligning views. For example, it's common to see
code like this:

```swift
HStack {
  Spacer()
  Text("Some right-aligned text")
}
```

Instead, use a flexible frame with alignment:

```swift
Text("Some right-aligned text")
  .frame(maxWidth: .infinity, alignment: .trailing)
```

This is preferred over an `HStack` and the `Spacer`, because there's an edge case
in the `HStack` solution. The `Spacer` has a default minimum length (equal to
the default spacing). As a result, the text might start wrapping or truncating
sooner than necessary, because the spacer also occupies some of the proposed
width of the `HStack`.

### View Modifiers

View modifiers always wrap an existing view inside another layer: the modifier
becomes the parent of the view it's applied to. While SwiftUI has the `.modifier`
API to apply a value conforming to the `ViewModifier` protocol, SwiftUI's
built-in modifiers are all exposed on `View`.

#### Padding

The `.padding` modifier uses its padding value to modify the proposed size by
subtracting he padding from the specified edges. This modified size is then
proposed to the subview of `.padding`.

By writing `.padding()` without an arguments, the default padding for the
current platform is used.

#### Fixed Frames

The fixed frame modifier, `.frame(width:height:alignment:)`, has very simple
layout behavior: it proposes exactly the specified size to its subview, and it
reports exactly the specified size as its own size, independent of the reported
size of its subview.

With support for dynamic type and various screen sizes, using fixed frames in
production code is actually quite rare. It's a good practice to avoid hardcoding
magic numbers in fixed frames unless absolutely necessary, e.g., for certain
design elements that intrinsically cannot or should not scale.

#### Flexible Frames

The API for flexible frames has a ton of parameters: the minimum, maximum, and
ideal values for both the frame's width and height. The behavior of flexible
frames isn't very intuitive, but they're a useful tool.

Flexible frames apply the minimum and maximum boundaries twice: once for the size
they propose to their subview, and once to determine their reported size.

The flexible frame takes the proposed size and clamps it by whatever minimum and
maximum parameters were specified. This clamped size is then proposed to the
frame's subview.

Once the subviews' size has been computed, the flexible frame determines its
own size based on the proposed size by applying the boundaries again.
However, now a missing boundary value – e.g., if we only specify `minWidth`,
but not `maxWidth` – will be substituted using the subview's reported size.
Then the proposed size gets clamped by these boundaries and reported as the
frame's size.

In practice, this means that if we only specify `minWidth`, the flexible frame
will becomes at least `minWidth`, and at most, the width of its subview. If we
only specify `maxWidth`, the flexible frame will become at least the width of
its subview, and at most, `maxWidth`. The same applies for the height.

The first common way of using flexible frames is like this:

```Swift
Text("Hello, World!")
  .frame(maxWidth: .infinity)
  .background(.quaternary)
```

The `.frame(maxWidth: .infinity)` pattern makes sure the flexible frame becomes
at least as wide as proposed, or the width of its subview if that's wider than
proposed. This is commonly used to create views that span the entire available
width. It's kinda like saying take the proposed width or greater.

Another common pattern in the following:

```Swift
Text("Hello, World")
  .frame(minWidth: 0, maxWidth: .infinity)
  .background(Color.teal)
```

This makes sure the frame always becomes exactly as wide as proposed, independent
of the size of its subview. It's kinda like saying take the proposed width.

Both patterns can be applied to the height as well. In the following example,
assuming the rendering screen size is 320⨉480:

```Swift
Text("Hello, World")
  .frame(minWidth: 0, maxWidth: .infinity)
  .background(Color.teal)
  .padding(10)

      |
    1 | 10
   padding
      |
    2 | 9
  background
      |
   -------
 3 | 6  7 | 8
 frame  Color
   |
 4 | 5
 Text
```

1. The system proposes 320⨉480 to the padding.
2. The padding proposes 300⨉460 to the background.
3. The background proposes  the same 300⨉460 to its primary subview (the frame).
4. The frame proposes the same 300⨉460 to its subview (the text).
5. The text reports its size as 76⨉17
6. The frame's width becomes `max(0, min(.infinity, 300)) = 300`. Note that
the `0` and `.infinity` values are the arguments specified for the flexible frame.
7. The background proposes the size of the flexible frame (`300⨉17`) to the
secondary subview (`Color`).
8. The color accepts and reports the proposed size.
9. The background reports the size of its primary subview (`300⨉17`).
10. The padding adds 10 point on each side, and it report its size as `300⨉37`.

The flexible frames APIs is the only one in SwiftUI to explicitly specify an
ideal size, i.e., the size that will be adopted if `nil` is proposed for one or
both dimensions. If the `idealWidth` or `idealHeight` parameters are specified,
this size will be proposed ot the frame's subview, and it'll also be reported as
the frame's own size, regardless of the size of its subview.

#### Aspect Ratio

In `Color.secondary.aspectRatio(4/3, contentMode: .fit)`, the `aspectRatio`
modifier will compute a rectangle with an aspect ratio of 4/3 that fits into the
proposed size and then proposes that to its subview. To the parent view, it always
reports the size of its subview, regardless of the proposed size or the specified
aspect ratio.

> The `scaleToFit` and `scaleToFill` modifiers are shorthand for
> `.aspectRatio(contentMode: .fit)` and `.aspectRatio(contentModel: .fill)`, respectively.

#### Overlay and Background

Layout-wise, they work exactly the same way. The only difference is that an
overlay draws the secondary view on otp of the primary view, whereas a
background draws the secondary view behind the primary view.

Background and overlay don't influence the layout of their primary subviews.
The reported size of the overlay or background is always the reported size
of the primary subview.

The background/overlay modifier always becomes the size of its primary subview,
independent of the view in the background (the secondary subview). So,
in the example below, the size of the `body` stays 30 by 30 while the
background view, due to the padding, draws a 40 by 40 view:

```Swift
var body: some View {
  Rectangle()
    .frame(width: 30, height: 30)
    .background { Color.red.padding(-10) }
    .border(.blue)
}
```

Within an overlay or background, when the secondary view is a view list
containing multiple views, these views are placed in an implicit `ZStack`.

#### Fixed Size

The `fixedSize()` modifier proposes `nil` for the `width` and `height` to its
subviews, regardless of its own proposed size. This way, the subview becomes
its ideal size.

One of the most common use cases for `fixedSize` is with `Text` views. Normally,
`Text` will do anything to render itself inside the proposed size (e.g., truncation
or word wrapping). One way to circumvent this is by proposing `nil⨉nil` to the test
so that it becomes its ideal size.

### Container Views

#### HStack and VStack

Horizontal and vertical stacks lay out their subviews in the same way, just with
a different major axis.

Consider the following stack:

```Swift
HStack(spacing: 0) {
  Color.red
  Text("Hello, World!")
  Color.teal
}
```

With a large-enough width, this renders as expected. But as the width gets
smaller, the the text  wraps, even though there's enough spacing to show it in full.

The reason for this behavior is how the `HStack` algorithm divides up the
available width among its subviews:

1. The stack determines the _flexibility_ of its subviews.
2. The stack then sorts the subviews according to _flexibility_, from least
flexible to most flexible. It keeps track of all the remaining subviews and
the available remaining width.
3. White there are remaining subviews, the stack proposes the remaining
width, divided by the number of the remaining subviews.

In the following `HStack`, the `Text` is the least flexible subview.

```Swift
HStack(spacing: 0) {
  Color.red
  Text("Hello, World!")
  Color.teal
}
```

Assuming the `Text` ideal width is 100, when 180⨉180 is proposed to the `HStack`,
it proposes 180/3 for the width to the least flexible subview - the `Text`. The
`Text` will then wrap or truncate as needed. Let's assume it becomes 50⨉50 points
(inserting a line break). The two rectangles then get proposed 130/2 and 65/1 points,
respectively. Because of this algorithm, even though there's enough space for
the text, it doesn't grow to display itself without line breaks.

There are a few workarounds. Using `fixedSize()` modifier on the `Text` will
make it ignore the proposed size and always becomes the ideal size. However,
once the `HStack` is proposed less than the ideal width of the text, the text
will still render at its full width and goes out of bounds.

Another alternative is to give the text a _layout priority_ using hte `.layoutPriority`
modifier. This will cause the `HStack` to first propose the full remaining width
to the text and then use whatever remains after that to the two colors.

```Swift
HStack(spacing: 0) {
  Color.red
  // Default value is typically 0.
  Text("Hello, World!").layoutPriority(1)
  Color.teal
}
```

#### ZStack

`ZStack` is different from an overlay or background. Overlay and background
take on the size of the primary subview and discard the size of the secondary
subview. `ZStack` uses the union of the subviews' frames to compute the
stack's own size.

In the example below, when the `ZStack` is used as the root view for a new
iOS application, the color stretches to the full size of the safe area. This
is because the `ZStack` gets proposed the entire safe area, and it takes that
proposal and proposes the same value to each of its subviews.

```Swift
VStack {
  Color.teal
  Text("Hello, world")
}
```

#### Scroll View

With scroll views, have to distinguish between the layout behavior of the actual
scroll view and the layout behavior of the scroll view's contents, which scrolls
within the visible area of the scroll view.

The scroll view itself accepts the proposed size along the scroll axis, and it
becomes the size of its content on the other axis. For example, if we place a
vertical scroll view as the root view, it'll become the height of the entire
safe area, and it'll take on the width of the scroll content.

Along the scroll axis, a scroll view essentially has unlimited space. Therefore,
the scroll view proposes `nil` in the scroll axis (or axes), and it proposes
the unmodified dimension for the other axis.

By default, the contents of the scroll view are placed inside an implicit `VStack`,
regardless of the scroll direction.

When we have a scroll view with a width less wide than the proposed width, our
scroll view becomes less wide than proposed. To fix this, we can add a
`.frame(maxWidth: .infinity)` to our subview (`Text` for instance).

When we put a shape into a scroll view, the built-in shapes in SwiftUI all have
an ideal size of 10⨉10. This is due to the default argument of `.replacingUnspecifiedDimension(by:)`
on `ProposedViewSize`, which is 10⨉10. Since the scroll view proposed `nil`
along the scroll axis, the shape will take on its ideal size of 10 points
on that axis. To change this, can specify an explicit height using
`.frame(height:)` or `.frame(idealHeight:)`, or we could use `.aspectRatio`.

#### Geometry Reader

Geometry reader are used to get access to the proposed size. It always accepts
the proposed size and reports that size to its view builder closure via a
`GeometryProxy`, giving us access to the geometry reader's size.

Because the geometry reader always becomes the proposed size, if we want to - for
example - measure the width of some `Text` view by putting a geometry reader
around it, this will influence layout around the text. Here's two good ways to
use `GeometryReader`:

- When we wrap a completely flexible view inside a `GeometryReader`, it won't
affect the layout. For example, when we have `ScrollView`, which becomes the
proposed size anyway, we can wrap it using a geometry reader to access the
proposed size.
- When we put a `GeometryReader` inside a background or overlay modifier, it
won't influence the size of the primary view. Inside the background or overlay,
we can then use the proxy to read out different values related to the view's
geometry. This is useful to measure the size of a view, as the size of the primary
subview will be proposed to the secondary subview (the geometry reader).

`GeometryReader` takes no alignment parameter and place their subviews top-leading
by default, whereas all the container views except the `ScrollView`, use center
alignment by default.

#### List

Equivalent of `UITableView` or `NSTableView`. The `List` itself takes on the
proposed size, and similarly to `ScrollView`, it proposes its own width, and `nil`
for the height, to its subview.

The sizing of the list's content in the vertical direction is complicated, because
hte list items are laid out lazily. Similar to `UITableView` with non-fixes row
heights, `List` estimates the entire height of the content based on the items that
have already been laid out.

#### LazyHStack and LazyVStack

LazyHStack and LazyVStack behave in the same way, just with different major and
minor axes. Lazy stacks share layout behavior with their non-lazy counterparts
insofar as they become the size of the union of all subviews' frames.

However, lazy stacks don't attempt to distribute the available space along the
major axis among their subviews. For example, a `LazyHStack` just proposes
`nil⨉proposedHeight` to its subviews, i.e., the subviews become their ideal width.

Computing the height of the lazy vertical stack is complicated by the fact that it
creates its subviews lazily when it's embedded inside a scroll view a scroll view
(for the width, the lazy vertical stack accepts the proposed width).

As we scroll, more and more print messages start to appear, and views in the render
tree get created as needed. Therefore, just like `List`, the `LazyVStack` needs to
estimate its height based on the subviews that have already been laid out and update
its own size as new subviews appear onscreen.

When choosing the type of stack view to use, always start with a standard stack
view and only switch to a lazy stack if profiling your code shows a worthwhile
performance improvement. For more information on lazy stack views and how to
measure your app’s view loading performance, see [Creating performant scrollable
stacks.](https://developer.apple.com/documentation/swiftui/creating-performant-scrollable-stacks)

> Never profile your code using the iOS simulator.
> Always use real devices for performance testing<sup>[*](https://developer.apple.com/documentation/swiftui/creating-performant-scrollable-stacks#Profile-to-find-performance-problems)</sup>.

Here's another quote from [Apple Developer forum](https://developer.apple.com/forums/thread/651593?answerId=616783022#616783022):

> Lazy stacks incur a small amount of extra overhead, both in time and memory,
> to handle the bookkeeping for what views have and have not been instantiated.
> In cases where all the views have to be instantiated anyway, that overhead is
> pure cost with no benefit.
>
> It’s also worth noting that all the views in a lazy stack must be instantiated
> in some situations that might be unexpected at first glance. For example consider
> a vertically scrolling `ScrollView` containing a `LazyVStack`. So far, so good.
> But now suppose that `LazyVStack` includes horizontally scrolling ScrollViews within
> it—think rows with photo thumbnails. In the general case, all the views in the
> horizontally scrolling `ScrollView` have to be instantiated eagerly so that SwiftUI
> can determine the height of the row.
>
> These unexpected cases are the reason to prefer regular stacks and switch to lazy
> stacks when it makes a measurable difference. If lazy stacks were always the right
> answer, SwiftUI could have just made all stacks lazy.
>
> Practically, if the highest level `ScrollView` in a view hierarchy directly wraps
> an `HStack` or `VStack`, then that’s a good candidate for a lazy stack. Beyond
> that, profile first lest you accidentally make your app slower while trying to
> make it faster.

There's also this recommendation from [WWDC20](https://developer.apple.com/videos/play/wwdc2020/10031?time=299):

> As a rule, if you aren't sure which type of stack to use, use `VStack` or `HStack`.
> Adopt Lazy Stacks as a way to resolve performance bottlenecks that you find
> after profiling with Instruments.

#### LazyVGrid and LazyHGrid

The first step of layout out a `LazyVGrid` is to compute the width of the columns
based on the grid's proposed width. There are three column types:

1. fixed columns
2. flexible columns
3. adaptive columns

Fixed-width columns unconditionally become their specified width, flexible columns
are flexible (but have lower and upper bounds on their width), and adaptive columns
are really containers that host multiple sub-columns.

The grid starts by subtracting all the fixed-column width and spacings from the
proposed width. For the remaining columns, the algorithm proposes the remaining
width, divided by the number of remaining columns, in order fo the columns.

Flexible columns clamp this proposal using their bounds (minimum and/or maximum
width).

Adaptive columns are special though: the grid tries to fit as many sub-columns
inside an adaptive column as possible by taking the proposed column width and
dividing it by the minimum width specified for the adaptive column. These
sub-columns can then be stretched out to the specified maximum width to fill
the remaining space.

The grid computes its own width as the sum of the column widths, plus any spacing
in between columns. For its height, the grid proposes `columnWidth⨉nil` to its
subviews to compute the row heights and then computes its own height as the
sum of the row heights plus spacing.

Contrary to `HStack` and `VStack`, grids go through the column layout algorithm
twice:

1. once during the layout pass
2. and then again during the render pass

During the layout, the grid starts out with its proposed width. However, during
the render pass, it starts out with the width that was calculated during the
layout pass, and it divides that width among the columns again.

```Swift
var body: some View {
  LazyVGrid(
    columns: [
      GridItem(.flexible(minimum: 60)),
      GridItem(.flexible(minimum: 120))
    ],
    spacing: 10,
    content: {
      Rectangle().fill(.blue.opacity(0.5))
      Rectangle().fill(.red.opacity(0.5))
    }
  )
  .frame(width: 200)
  .border(.teal)
}
```

In the example above, the grid renders out of bounds and also renders off-center
of its enclosing 200-point-wide frame. This is because the grid starts with width
of 200 points, minus 10 points of spacing. For the first column, we calculate the
width as 190 / 2, which equals 95 points. Since the first column has a minimum
width of 60p, the width of 95p isn't affected by the clamping, so the remaining
width stands as 95p. The second column becomes 95p clamped to its minimum of 120p,
i.e., 120p wide.

However, the gird renders the first column at 108p wide, and the second one at
120p wide. That's where the second layout pass comes in.

The overall width of the grid as calculated by the first pass is
`95 + 10 + 120 = 225` points. The fixed frame width of 200 points around the
grid centers the grid, shifting it `(225 - 200) / 2 = 12.5` points to the left.
When it's time for the grid to render itself, it goes through the column layout
again, but this time starting with a remaining width of 225p.

The second pass starts with 225p minus 10p spacing, or 215p. The first column
becomes `215 / 2 = 108`. The remaining width is now `215 - 108 = 107`. The
second column becomes 107p clamped to its minimum of 120p. This is exactly what
we see in the example above.

Since the frame around the grid has calculated the grid's origin based on the
original width of 225p, but the grid now renders itself with an overall width
of `108 + 10 + 120 = 238p`, the grid appears out of center by about 7p<sup>[*](https://www.objc.io/blog/2020/11/23/grid-layout/)</sup>.

#### ViewThatFits

When we want to display different views depending on the proposed size, we
can use `ViewThatFits`. It takes a number of subviews, and it displays the
the first subview that fits. It does so by proposing `nil` to figure out
the ideal size for each subview, and it displays the first (in the order
the subviews appear in the code) where the ideal size fits within the proposed
size.

If none of the subviews fit, it picks the last subview.

### Rendering Modifiers

Modifiers that influence how a view is rendered, but that don't influence
the layout itself - for example, `offset`, `rotationEffect`, `scaleEffect`,
etc.

We can think of these modifiers as performing something like a `CGContext.translate`
to modify where a view is drawn. However, from the layout system's perspective,
the view is still situated in its original position.

#### Alignment

SwiftUI uses alignment to position views relative to each other. By default,
all views center their subviews.

When the alignment is center, the parent view asks the subview for its horizontal
and vertical center, the values that the subview returns are called _horizontal
center alignment guide_ and _vertical center alignment guide_, respectively.

The most important point to understanding SwiftUI's alignment system is that
alignment is always determined in "negotiation" between the parent and the
subview.

#### Modifying Alignment Guides

Note that the `Alignment` type isn't the alignment guide. Instead, it's what
determines _which_ alignment guide to use.

Using the built-in alignment guides, we can only align multiple views using
the same alignment for each view. For example, align the top edge of one view to
the top edge of another view. However, we cannot align e.g., the center of
one view to the top trailing corner of another view. Unless we override the
built-in (implicit) alignment guides and even create our own alignments.

The view can provide an _explicit_ alignment guide for a certain alignment.
For example, we can override how the first text baseline is computed for an
image:

```swift
var body: some View {
  let image = Image(systemName: "pencil.circle.fill")
    .alignmentGuide(.firstTextBaseline, computeValue: { dimension in dimension.height / 2 })
  
  HStack(alignment: .firstTextBaseline) {
    image
    Text("Pencil")
  }
}
```

However, if we change the stack alignment to `.center`, our custom alignment
guide doesn't affect the alignment.

The parameter passed to the `computedValue` closure is of type `ViewDimensions`.
This is similar to a `CGSize` in that it has a width and height, but it also
allows us to access the underlying view's alignment guides.

In this example, we want to overlay the center of a badge view on the top
trailing corner of another view:

```swift
extension View {
  func badge<B: View>(@ViewBuilder _ badge: () -> B) -> some View {
    overlay(alignment: .topTrailing) {
      badge()
        .alignmentGuide(.top) { $0.height / 2 }
        .alignmentGuide(.trailing) { $0.width / 2 }
    }
  }
}
```

#### Custom Alignment Identifiers

Consider the layout for this view:

```Swift
struct CustomAlignmentView: View {
  var body: some View {
    VStack() {
      HStack {
        Text("Inbox")
        CircleButton(symbolName: "tray.and.arrow.down")
          .frame(width: 30, height: 30)
      }
      HStack {
        Text("Sent")
        CircleButton(symbolName: "tray.and.arrow.up")
          .frame(width: 30, height: 30)
      }
      HStack {
        CircleButton(symbolName: "line.3.horizontal")
          .frame(width: 40, height: 40)
      }
    }
  }
}

private struct CircleButton: View {
  let symbolName: String
  
  var body: some View {
    Image(systemName: symbolName)
       .resizable()
      .foregroundColor(.black)
      .padding()
      .background(Color.gray)
      .clipShape(Circle())
  }
}
```

The problem is the horizontal alignment of the `VStack`. We want the `CircleButton`
to be vertically stacked on each other with a center alignment while the `Text` are
stacked on top of each other with a trailing alignment:

```text
Inbox   <Icon-1>
 Sent   <Icon-2>
       <Icon---3>
```

We could try to specify `.trailing` alignment on the `VStack` and then specify an
explicit alignment guide for `.trailing` on both items' `CircleButton`. However,
this doesn't work either. When `VStack` places its subviews, it asks each subview
for its `.trailing` alignment guide. Since `HStack` has its own `.trailing`
alignment guide, the explicit alignment guide on `CircleButton` will never be used.

Custom alignments can propagate up through multiple container views. First thing,
need a type conforming to the `AlignmentID` protocol:

```Swift
struct MenuAlignment: AlignmentID {
  static func defaultValue(in context: ViewDimensions) -> CGFloat {
    context.width / 2
  }
}
```

The default value that `defaultValue(in:)` returns is used unless we specify
the view's horizontal center as the default value. Now we can define a static
constant for our custom alignment on the `HorizontalAlignment` struct, just
as SwiftUI does for the built-in alignments:

```Swift
extension HorizontalAlignment {
  static let menu = HorizontalAlignment(MenuAlignment.self)
}
```

Now we can use it in the menu `VStack` and specify explicit alignment guides for
`.menu` on the CircleButtons:

```Swift
struct CustomAlignmentView: View {
  var body: some View {
    VStack() {
      HStack {
        Text("Inbox")
        CircleButton(symbolName: "tray.and.arrow.down")
          .frame(width: 30, height: 30)
          .alignmentGuide(.menu) { $0.width / 2 }
      }
      HStack {
        Text("Sent")
        CircleButton(symbolName: "tray.and.arrow.up")
          .frame(width: 30, height: 30)
          .alignmentGuide(.menu) { $0.width / 2 }
      }
      HStack {
        CircleButton(symbolName: "line.3.horizontal")
          .frame(width: 40, height: 40)
      }
    }
  }
}
```

The last `CircleButton` doesn't need the `.menu` alignment because the default
value of `.menu` is already the center.

When the `VStack` asks its subviews for the `.menu` alignment guide, the `HStack`
will consult their subviews for an explicit alignment guide value before falling
back tot he default value. This means that the `HStack` will return the explicit
alignment guide we specified on the `CircleButton` when asked for its `.menu`
alignment guide. In other words, the subview's explicit alignment guide is used
instead of the stack's implicit alignment guide.
