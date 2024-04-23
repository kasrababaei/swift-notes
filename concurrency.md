# Swift Concurrency

- [Tasks](#tasks)
- [Isolation](#isolation)
  - [Types of isolations](#types-of-isolations)
- [Concurrency-safe Singletons](#concurrency-safe-singletons)

When to switch to unstructured concurrency:
Careful with withCheckedContinuation. If the block of code isn’t that big, you’re just converting into Swift concurrency, and converting out of Swift concurrency. What’s the point? Probably should start with the network calls.

★ Need to have the strict concurrency turned on, otherwise you’re not getting that payoff.

In a large app, try to identify a subsystem that is large enough and important enough to warrant the effort to do so. Just leave that subsystem, tackle issues before going to convert more subsystems.

The URLSessionDataTask itself is cancellable. If your task gets cancelled, the URLSession itself cancels the data task.

In `withTaskCancellationHandler`, you want to make sure that either the cancellation block wins or the operation block. You don’t cancel it if it’s been completed already, and if you cancelled it, you’d ignore the completion. You’ll need some sort of concurrency data shared structure. The cancellation handler is called in an arbitrary context. There’s no guarantee in what context that’ll get called. So, any work done in there, has to be serialized with some sort of lock vs the work that’s done in the operation block.

When working with a subsystem that is very tied to dispatch queue, it’s natural to use a dispatch queue for the cancellation. When the callback is happening on a queue, and you’re telling it what queue to call it and what queue to run the callbacks, better to just stick to using dispatch queue and use the natural serialization associated with that.

Better use `OSAllocatedUnfairLock` but it’s only available in iOS 16. It’s fast, has no obj-c overhead, and uses inline attribute. Only use DispatchQueue serialization if you’re already in the dispatch queue world.

The act of confinement is determined statically at compile time because that’s why the compiler can check it at compile time. The task initializer has a hidden parameter that causes the inheritance but they’re tiding it up right now.

In SwiftUI, only the view’s body is isolated to the main actor. If you define a method outside of the body, then that won’t be isolated on the main actor.

Nothing’s wrong with running things on the main actor as long as you are careful with APIs that block it when you don’t expect it. Keychain is a good example.

Actors are made to guard mutable states.

??? Make core of it into an asynchronous function that’s no actor bound.

`withCheckedContinuation` is the glue code between Swift concurrency world to other concurrency domains such as dispatch queue world.

`Task.detach` is for running operations asynchronously as part of a new top-level task.

## Tasks

A task is the basic unit of concurrency in the system. Every asynchronous function is executing in a task. In other words, a task is to asynchronous functions, what a thread is to synchronous functions.

## Isolation

Isolation is the mechanism that Swift uses to make data races impossible. Isolation is specified at compile time. Everything is non-isolated by default. You must take explicit action to change this. The isolation behavior is still controlled by the definition. Isolation will not suddenly change unless you decide you want to change it.

- Rule 1: isolation is controlled by definitions
- Rule 2: isolation can only change via async calls. Synchronous code cannot change isolation.
- Rule 3: protocols can specify isolation

Whatever isolation was in effect when you create a Task will still be used inside the Task body by default. 

```Swift
@MainActor
class MyIsolatedClass {
    func myMethod() {
        Task {
            // still isolated to the MainActor here!
        }
        
        Task.detached {
            // explicitly non-isolated, regardless
            // the enclosing scope
        }
    }
}
```

#### Types of isolations

- None
- Static: actor types, global actors (like `@MainActor`), and isolated parameters
- Dynamic: It can happen that the type system alone does not or cannot describe the isolation actually used

If something has isolation that you don’t want, you can opt-out with the nonisolated keyword. This also can make a lot of sense for static constants that are immutable and safe to access from other threads.

Protocols, being definitions, can control isolation just like other kinds of definitions.

```Swift
protocol NoIsolation {
    func method()
}

@MainActor
protocol GloballyIsolatedProtocol {
    func method()
}

protocol PerMemberIsolatedProtocol {
    @MainActor
    func method()
}
```

It is possible to apply isolation to the protocol as a whole:

```Swift
@MyCustomNonMainGlobalActor
protocol CustomActorStuff {
    ...
}

// This does not affect the isolation of the UIViewController
// type. It's still MainActor...
extension UIViewController: CustomActorStuff {
    // ... but everything in here will be inferred
    // to use MyCustomNonMainGlobalActor
}
```

Dynamic isolation comes up regularly with systems built before concurrency was a thing. One tool we have to deal with this is dynamic isolation. These are APIs that allow us to express isolation in a way that is invisible by just looking at definitions.

```Swift
// this delegate will actually always make calls on the MainActor
// but its definition does not express this.
@MainActor
class MyMainActorClass: SomeDelegate {
    nonisolated funcSomeDelegateCallback() {
        // promise the compiler we will be on the
        // MainActor at runtime.
        MainActor.assumeIsolated {
            // access MainActor stuff, including self
        }
    }
}
```

Isolation cannot change at all for a synchronous function. And it cannot change for the synchronous parts of an async function. But, as soon as you need to make an async call, it could.

This is really a consequence of the fact that isolation is controlled entirely by a function’s definition. It does not matter how the caller is being isolated. This is completely different from how queues or locks work.

#### Concurrency-safe Singletons

There's three options:

- Make the type conform to `Sendable`. Ideally, this needs to be placed in the same file that the class was declared. Otherwise, need to use `@unchecked Sendable` for retroactive conformance.
- Isolate the class by a global actor such as `@MainActor`, making its access serialized and concurrency-safe. However, actor isolation might not always work since it complicates access from non-concurrency contexts.
- Using `nonisolated(unsafe)` keyword
