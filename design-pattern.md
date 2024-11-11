# Design Pattern

- [Design Pattern](#design-pattern)
  - [Model-View-Controller (MVC)](#model-view-controller-mvc)
  - [Model–View–ViewModel (MVVM)](#modelviewviewmodel-mvvm)
  - [Dependency Injection](#dependency-injection)
    - [Initializer Injection](#initializer-injection)
    - [Property Injection](#property-injection)
    - [Argument Injection](#argument-injection)
    - [Environment (or Context)](#environment-or-context)
    - [DI Framework](#di-framework)
    - [Instance Creation](#instance-creation)
  - [Key-Value Observation (KVO)](#key-value-observation-kvo)
  - [Singletons](#singletons)

In software engineering, a design pattern describes a relatively small,
well-defined aspect (i.e. functionality) of a computer program in terms of
how to write the code.

Using a pattern is intended to leverage an existing concept rather than re-inventing
it. This can decrease the time to develop software and increase the quality of
the resulting program.

Notably, a pattern does not consist of a software artifact. Most development
resources that a programmer uses involve configuring the codebase to use an
artifact; for example a library. In contrast, to use a pattern, a programmer
writes code as described by the pattern. The result is unique every time even
though the result may be recognizable as based on the pattern.

When choosing an architecture, must consider things such as:

- Project requirements: Consider whether the project will need scalability,
maintainability, testability, or flexibility in the future.
- Complexity and Simplicity: Strive for a balance between simplicity and
functionality. Overly complex patterns can make code more difficult to understand
and maintain.
- Testing Requirements: If unit testing is a priority, consider patterns that
promote testable code.
- Flexibility and Reusability: Some patterns provide better flexibility for code
reuse and modification. Patterns like Decorator or Strategy make it easier to
add or modify behaviors without extensive code changes.
- Team Experience and Consistency: Consider the team's familiarity with certain patterns.
- Scalability and Performance: Some patterns can be more demanding on resources,
so consider the performance needs of your app.

You can start by asking questions like:

- How dependency injection work
- How does testing work which goes hand-to-hand with dependency injection
- How does routing (for deep links) work
- Where does the business logic live

## Model-View-Controller (MVC)

It is a high-level pattern in that it concerns itself with the global
architecture of an application and classifies objects according to the general
roles they play in an application. It is also a compound pattern in that it
comprises several, more elemental patterns. The MVC design pattern considers
there to be three types of objects: model objects, view objects, and controller
objects<sup>[*](https://developer.apple.com/library/archive/documentation/General/Conceptual/CocoaEncyclopedia/Model-View-Controller/Model-View-Controller.html#//apple_ref/doc/uid/TP40010810-CH14-SW1)</sup>.

- Model: model objects encapsulate data and basic behaviors.
- View: view objects present information to the user.
- Controller: controller objects tie the model to the view.

## Model–View–ViewModel (MVVM)

The Model-View-ViewModel (MVVM) design pattern to create a clear separation between
UI and business logic, resulting in modular, testable, and maintainable code.

- Model: represents the core data or business logic of the application. It contains
  the data structure and any operations or transformations related to the data.
  This layer should be isolated and independent of any UI code to keep business
  logic separate from presentation logic​.
- View: this layer includes the user interface, typically implemented through
SwiftUI views or UIKit components, depending on the framework. It doesn’t
directly interact with the Model. Instead, it relies on the ViewModel to mediate
data changes and interactions​.
- ViewModel: Acts as the intermediary between the Model and the View. The
ViewModel transforms data from the Model into a format that is easy for the View
to display. It also interprets user interactions from the View, updating the Model
as needed.

In MVVM, the responsibility for fetching data from the BE and mapping
it to models typically falls within a *service layer* or *repository layer* outside
of the Model-View-ViewModel triad. This layer interacts with the backend, parses
the data into model objects, and provides them to the ViewModel.

## Dependency Injection

Dependency injection in short means dependencies are handed instead of reaching
for them.

It is a programming technique in which an object or function
receives other objects or functions that it requires, as opposed to creating
them internally. Dependency injection aims to separate the concerns of
constructing objects and using them, leading to loosely coupled programs.
The pattern ensures that an object or function that wants to use a given service
should not have to know how to construct those services. Instead, the receiving
"client" (object or function) is provided with its dependencies by external code
(an "injector"), which it is not aware of. Dependency injection makes
implicit dependencies explicit. In short, dependency injection is done to achieve:

- Isolation: the code for creating the dependency lives in one place.
- Reusability: goes hand-to-hand with isolation. More places can use a common
interface that hands over the dependency.
- Testability:
  - Tests can inject mocks in place of real dependency
  - Exercise type under test in isolation; so that your test isn't testing the
  entire codebase
  - Changes to other types shouldn't break your test

When picking an approach, note that there's no "one size fits all" solution.
Evaluate your needs, priorities and preferences. For instance, think about
decentralization, compile-time safety, thread safety or the incremental path.

Some of the common ways to do dependency injections:

### Initializer Injection

It is simple and straightforward. The passed value would be immutable. But there's
a "superset" problem. Parents must have superset of all their descendants' dependencies.
Lastly, challenging for foundational dependencies (e.g., storage, networking,
and analytics). Lastly, static functions, protocols and extensions don't have initializer.

The compile-time safety means there's no need to deal with codes like this and
hope at that at runtime it won't crash:

```Swift
func resolve<T>(...) -> T?
```

### Property Injection

Same problems as [Initializer Injection](#initializer-injection), but it can work
for static members (but not protocol extensions). But the dependency now has to
be exposed, i.e., either internal or public. Lastly, sometimes it has to be either
optional or implicitly unwrapped optional.

### Argument Injection

This works for static functions, protocol extensions and non-OOP code as well.
But as API boilerplate (how many parameters are we going to pass into it) and
has the same "superset" problem that was explained in [Initializer Injection](#initializer-injection).

### Environment (or Context)

Like one singleton to rule them all right. This avoids the superset problem, and
has a minimal boilerplate. However, it doesn't scale nicely. It works for a few
things but when there's thousands of dependencies, it'll become a big environment.
Also, an environment that knows all about these types, those types can depend on
things that cause a circular import cycle in your modular structure.

```Swift
func update() {
  Environment.rideAPI.sendUpdate(...)
}
```

### DI Framework

These framework requires someone to work on them. But once it's built, i'll just
work. Handles circular dependencies, thread safety, compile-time safety, and
scoping like local scopes, graphs. singletons.

But it brings in some complexity and requires hundreds or thousands lines of code.
There's a hard learning curve. Often makes it harder to reason about the production
code.

Incremental adoption can be difficult and it's just one dependency to rule them all.

### Instance Creation

The goal is to separate business logic from instance creation responsibility.
This aligns well with the *Single Responsibility Principle*.

This approach is based on the principle provided by [Lyft](https://www.youtube.com/watch?v=dA9rGQRwHGs).
A DI system, at a high level, creates an instance like this:

```Swift
() -> T
```

However, this code doesn't state how to create an instance; so, let's add that:

```Swift
(() -> T) -> () -> T
```

 The first closure is actually creating the instance and returning a concrete
 type whereas the second one returns an interface. But then the compiler might
 return the same closure again. So, the compiler needs to know what type is
 expected to get back, which is the abstract interface:

 ```Swift
bind<T>(_ type: T.Type, to instantiator: @escaping () -> T) -> () -> T
 ```

## Key-Value Observation (KVO)

Key-value observing is a Cocoa programming pattern you use to notify objects about
changes to properties of other objects. It’s useful for communicating changes
between logically separated parts of your app—such as between models and views.
You can only use key-value observing with classes that inherit from `NSObject`.

Properties that are marked with both `@objc` and `dynamic`, can be observed using
KVO:

```Swift
class MyObjectToObserve: NSObject {
  @objc dynamic var myDate = NSDate(timeIntervalSince1970: 0)
  func updateDate() {
    myDate = myDate.addingTimeInterval(Double(2 << 30))
  }
}
```

An instance of an observer class manages information about changes made to one
or more properties.

```Swift
class MyObserver: NSObject {
  @objc var objectToObserve: MyObjectToObserve
  var observation: NSKeyValueObservation?
  
  init(object: MyObjectToObserve) {
    objectToObserve = object
    super.init()
    
    observation = observe(
      \.objectToObserve.myDate,
       options: [.old, .new]
    ) { object, change in
      print("OldValue: \(change.oldValue!), NewValue: \(change.newValue!)")
    }
  }
}
```

## Singletons

The Singleton is a design pattern that restricts a class to a single instance,
ensuring only one object of the class exists throughout the application. It’s
often used for shared resources like configurations, logging, network sessions,
or databases, where multiple instances could cause inconsistencies or conflicts.

Some cons of using singletons are:

- **Global State Dependency:** Global access can make testing and debugging
difficult, as parts of the code become dependent on shared state, leading to
tight coupling.
- **Limited Flexibility:** Since only one instance exists, replacing it for
testing or extending it for other purposes can be challenging.
- **Hidden Dependencies:** Singleton introduces hidden dependencies, as code
relying on a singleton may indirectly depend on the singleton’s state, making
code behavior less predictable.
- **Multithreading Issues:** Without proper thread-safety mechanisms, singletons
can cause race conditions in multithreaded contexts.
