# SwiftUI

> Views are funcitons of state, not of a sequence of events.

- [SwiftUI](#swiftui)
  - [View Builders](#view-builders)
  - [Group](#group)
  - [Dynamic Views vs Statick Views](#dynamic-views-vs-statick-views)
  - [Lifetime](#lifetime)
  - [Identity](#identity)
  - [State](#state)
  - [StateObject](#stateobject)
  - [ObservedObject](#observedobject)
  - [Binding](#binding)

## View Builders

A SwiftUI syntax for constructing list of views, which is built on top off Swift's result builder.

## Group

Group is a layout-agnostic abstractoin around a view builder.
When placing a `Group`, inclusing the modifers, as the root view or as the only subview within a scroll view, the group behaves like a `VStack` and the modifiers aren't applied to each individual view within the group. When placing a `Group` within an `Overlay` or `Background`, it behaves like an implicit `ZStack`.

## Dynamic Views vs Statick Views

```Swift
HStack {
    Text("Hello") // Static view
    if showText {
        Text(", World!") // dynamic view
    }
}

// View Tree:

        Hstack
          |
          |
    -------------
    |           |
    |           |
   Text       Text?
```

SwiftUI has something called the `attribute graph`, which includes more than just the rendered views; it also contains the state and tracks dependencies. Apple calls the nodes in the render tree `attributes`.

`Nodes` in a render tree have a lifetime which is not bound to whether the view is onscreen or not. For example, a `LazyVStack` renders lazily as opposed to a `VStack` that does it _eagerly_, still, the nodes in the render tree are preserved when they go offscreen. Bottom line is, we have no control over the lifetime.

## Lifetime

SwiftUI provides three hooks into lifetime events:

- `onAppear`: can be called multiple times
- `onDisappear`: the counter part of an `onAppear`
- `task`: combination of the two used for asynchronous work. This modifier creates a new `Task` at theh point where `onAppear` would be called, and it cancels this `Task` when `onDispapear` would be invoked.

## Identity

_Implicit identity_ is the position in the view tree which is used by SwiftuI Since it doesn't have reference types.

```Swift
HStack {
    Image(systemName: "hand.wave")
    if let g = greeting {
        Text(g)
    } else {
        Text("Hello")
    }
}

// This is just a humand-frendly model to demonstrate

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

Views can also have an _explicit identity_. This is mostly for views in a `ForEach` where each item is assigned an explicit identifier. For example, a unique identifier of the underlying data (either by conforming the items to the `Identifiable` protocol or by providing a `KeyPath` to a unique identifier). However, we can also assign explicit identifiers using the id modifier.

```Swift
var body: some View {
    Text("Hello").id("")
}
```

The id parameter can be any `Hashable` value.

Try not using patterns that create conditional content/branch in the view tree that might have unforseen consequences. Instead, use the following pattern:

```Swift
HStack { 
    Image(systemName: "hand.wave")
    Text("Hello")
        .background(highlightes ? .red : .clear)
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

As of iOS 17, SwiftUI doesn't depend on `Combine` and instead uses macro-based solution. `@State` property wrapper is now used for values and objects, whereas we usually only used it for values pre-iOS 17.

`State` is meant to be used for private view state values. During the execution of the body property, SwiftUI notices the state property is accessed; so, it adds a dependency between the state property and the view's node in the render tree.

If the state property is not rererenced inside the `body`, SwiftUI won't rerender the `body` when the state property changes.

Here's how we could write the same code without using the `@State` property wrapper:

```Swift
struct Counetr: View {
    private var _value: State(initialValue: 0)
    private var value: Int {
        get { _value.wrappedValue }
        nonmutating set { _value.wrappedValue = newValue }
    }

    var body: some View {
        Button("Incremenet: \(value))") {
            value += 1
        }
    }
}
```

The `State(initialValue:)` initializer makes it clear that the value 0 is just the _initial_ value of the state property. This is the value that will be used when the node for the counter view is first created in the render tree. Once the node is there, the initial value of the state property will be ignored.

## StateObject

The `@StateObject` property wrapper works much in the same way as @State: we specity an initial value (an object in this case), which will be used as the starting point when the node in the render tree is created. From then on, SwiftUI will kepp this object around across renders for the lifetime oft he node in the render tree. It  also observes the object for changes via the `ObservableObject` protocol.

`Observable` framework in iOS 17 replaced the entire existing object-observarion model based on the `Combine` framework. The `Observable` macro does two things:

1. adds conformance to the `Observable` marker protocol
2. changes object's properties to track both read and write access.

The macro only forms a dependency between the view and access properties. This is more efficient.

The new `@Observation` macro is more efficient than using the `@ObservableObject` property wrapper. If you only use one proerty of an object in your view's body, changes to the other properties won't cause redraws of this view. This can reduce unnecessary view updates and thus improve performance. Also, observable objects can be nested in optionals, arrays, or other collecitons. Observation and view updates will continue to work as expected, because of the property-level tracking of dependencies in the view's body.

Two common _mistakes_ with regard to observing objects:

1. to use `@State` when the lifetime of an object is managed somewhere outside of the view.
2. not using `@State` when an object's lifetime isn't managed somewhere outside of the view.

In the example below, the problem is that at the time the initializer runs, the view doesn't yet have _identity_. So, all this initializer does is change the initial value of the `@State` proeprty, but doesn't affect the state that's being used when the view is already onscreen.

```Swift
strctu Counter: View {
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
final class Counter: ObservableObjcet {
    var value: Int {
        willSet { objectWillChange.send() }
    }
}
```

If we implement our own `objectWillChange` publisher for some reason, `@Published` doesn't work.

The `Counter` view is created, but value of the state isn't present yet, because the initial value is stored as an `autoclosure`:

```Swift
struct CounterView: View {
    @StateObject private var model = Model()

    var body: some View {
        Button("Incremenet: \(model.value))") {
            model.value += 1
        }
    }
}
```

## ObservedObject

The `@observedObject` property wrapper is much simpler than `StateObject`: it doesn't have the concept of an initial value, and it doesn't maintain the observed object across renders. All it does is subscribe to the object's `objectWillChance` publisher and rerender the view when this publisher emits an event. This makes `ObservedObject` the only correct tool if we want to explicitly pass objects from the outside into a view (when targeting platforms before iOS 17). This is the equivalent of an Observable object within a regular property.

## Binding

When writing components, it's often the case that we need read and write access to a value, but don't need to know what the source of truth for this value actually is. Bindings are used exactly for this pupose, as we can see with many of SwiftUI's built-in components.
