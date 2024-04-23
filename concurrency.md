# Swift Concurrency

- [Swift Concurrency](#swift-concurrency)
  - [Tasks](#tasks)
    - [Task priority and cancelation](#task-priority-and-cancelation)
    - [Current task](#current-task)
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

A task is the basic unit of concurrency in the system. 

Every asynchronous function is executing in a task. In other words, a task is to asynchronous functions, what a thread is to synchronous functions.

So tasks seem to be capable of solving the thread explosion problem by using a pool of threads, all without having to manage an auxiliary object like an operation queue or dispatch queue. That’s already an improvement over threads and queues. Therefore, running the following code won't create a thousands thread:

```Swift
for n in 0..<workCount {
  Task {
    // It won't go more than 10-15 threads (depends on the computation power)
    print(n, Thread.current)
  }
}
```

Tasks also have the ability to schedule work after waiting some time. Threads accomplished this with a `sleep` function that was unfortunately blocking and so would tie up thread resources, and queues accomplished this by scheduling work to be executed on a future date. It’s not super ergonomic to have to use nanoseconds. Although this method is called “sleep”, it is quite different from the sleep we have used many times on Thread. This sleep does pause the current task for an amount of time, but it does not hold up the thread. The sleep function on Task works cooperatively so that other tasks can have a chance to do their work on the cooperative thread pool, and then once the time passes our task will be resumed.

We can spin up 1,000 tasks and sleep them for a second:

```Swift
for n in 0..<workCount {
  Task {
    try await Task.sleep(nanoseconds: NSEC_PER_SEC * 1000)
  }
}
```

When we await the sleep function we are able to suspend the execution of this task, thus freeing up its thread to be used by other tasks. If we run this and pause the executable we will see a small number of threads have been created, but they all have just a single stack frame with a cryptic function name:

```
Thread 2#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 3#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 4#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 5#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 6#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 7#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 8#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 9#0        0x00000001a05ef604 in __workq_kernreturn ()
Thread 10#0       0x00000001a05ef604 in __workq_kernreturn ()
Thread 11#0       0x00000001a05ef604 in __workq_kernreturn ()
Thread 12#0       0x00000001a05ef604 in __workq_kernreturn ()
```

his `__workq_kernreturn` function is how Grand Central Dispatch parks a thread while it waits for work to be scheduled on it. Once the suspension is over, the work can be resumed on any thread not just the one that started off of.

### Task priority and cancelation

Like all other forms of concurrency discussed so far, tasks also support the concept of priority, and it’s a granular value of finitely many possibilities:

```Swift
Task(priority: .low) {
  print("low")
}
Task(priority: .high) {
  print("high")
}

// prints out:
// high
// low
```

This allows you to tell the system how important this task is, and the runtime may be able to give this task more or less time depending on various factors.

In order to make use of cancellation we need to get a handle on the actual task we want to cancel, which means we need to assign a variable:

```Swift
let task = Task {
  print(Thread.current)
}
```

And then at any time we can cancel it:

```Swift
task.cancel()
```

However, if we run this we will see that the print statement still executes even though we canceled:

```Swift
<NSThread: 0x106204290>{number = 2, name = (null)}
```

This is a little different from threads, in which if you cancelled it right after creating it we could short-circuit the thread being started. Now we don’t know this is 100% always the case, but it seemed pretty consistent. So it seems tasks are a bit more eager in that they start up even if we cancel right after creating them.

The real way to short-circuit the work of a cancelled task is to manually check if the task has been cancelled:

```Swift
let task = Task {
  guard !Task.isCancelled
  else {
    print("Cancelled!")
    return
  }
  print(Thread.current)
}
task.cancel()
```

Even though `isCancelled` seems like a global value it is actually local to just the task we are operating in. This is similar to how cancellation worked with regular threads too.

### Current task

There is no Task.current static like there was `Thread.current`. This is because it’s possible to not be operating inside the context of a task, whereas you are always operating within a thread no matter what.

There is still a way to get the current task, but you have to invoke a function with a closure and that closure is handed the current task if it exists:

```Swift
Task {
  withUnsafeCurrentTask { task in
    print(task) // Optional(Swift.UnsafeCurrentTask(_task: (Opaque Value)))
  }
}
```

If your asynchronous context happens to also be a failable context, then you can check for cancellation via a throwing function rather than a boolean:

```Swift
let task = Task {
  try Task.checkCancellation()
  print(Thread.current)
}
task.cancel()
```

If the task has been cancelled when we try to invoke `checkCancellation`, the function will throw, thus short-circuiting the rest of the function. This can be a lot more ergonomic to invoke than guarding for `Task.isCancelled`.

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

### Types of isolations

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

## Concurrency-safe Singletons

There's three options:

- Make the type conform to `Sendable`. Ideally, this needs to be placed in the same file that the class was declared. Otherwise, need to use `@unchecked Sendable` for retroactive conformance.
- Isolate the class by a global actor such as `@MainActor`, making its access serialized and concurrency-safe. However, actor isolation might not always work since it complicates access from non-concurrency contexts.
- Using `nonisolated(unsafe)` keyword
