# Swift Concurrency

- [Swift Concurrency](#swift-concurrency)
  - [Operation Queue - iOS 2](#operation-queue---ios-2)
  - [Grand Central Dispatch (GCD) - iOS 8](#grand-central-dispatch-gcd---ios-8)
  - [Tasks - iOS 13](#tasks---ios-13)
    - [Task priority and cancellation](#task-priority-and-cancellation)
    - [Current task](#current-task)
    - [@TaskLocal](#tasklocal)
  - [Sendable Types vs Sendable Values](#sendable-types-vs-sendable-values)
  - [Isolation](#isolation)
    - [Dynamic Actor Isolation (Under-Specified Protocol)](#dynamic-actor-isolation-under-specified-protocol)
    - [Types of Isolation](#types-of-isolation)
      - [Explicit isolation](#explicit-isolation)
      - [isolated(any)](#isolatedany)
      - [Isolation inheritance](#isolation-inheritance)
      - [`sending` parameter and result values](#sending-parameter-and-result-values)
    - [Passing non-sendable types into actor-isolated context](#passing-non-sendable-types-into-actor-isolated-context)
  - [Concurrency-safe singletons](#concurrency-safe-singletons)
  - [Actor](#actor)
  - [@MainActor](#mainactor)
  - [Testing](#testing)

## Operation Queue - iOS 2

A queue that regulates the execution of operations. An operation queue invokes
its queued Operation objects based on their priority and readiness. After you
add an operation to a queue, it remains in the queue until the operation finishes
its task. You can’t directly remove an operation from a queue after you add it.

```Swift
let queue = OperationQueue()

queue.addOperation { print("1", Thread.current) }
queue.addOperation { print("2", Thread.current) }
queue.addOperation { print("3", Thread.current) }
queue.addOperation { print("4", Thread.current) }
queue.addOperation { print("5", Thread.current) }
```

Similar to [Threads](https://developer.apple.com/documentation/foundation/thread#),
the sample code above would create five different threads and runs the operations
in a non-sequential order. Also, similar to Threads, you can add priority on an
operations, called priority of service. To do so, you'll need a handle on the actual
operation before handing it off to the queue; and to do that we'll need to construct
an instance of the operation class. Although one can subclass it, Apple frameworks
ship with some convenience subclasses such as `BlockOperation`.

```Swift
// Adjusting the priority of service on an operation
let queue = OperationQueue()
let operation = BlockOperation {
  print(Thread.current)
}
operation.qualityOfService = .background
queue.addOperation(operation)
```

Canceling an operation object leaves the object in the queue but notifies the
object that it should stop its task as quickly as possible. For currently executing
operations, this means that the operation object’s work code must check the
cancellation state, stop what it is doing, and mark itself as finished.

```Swift
// Cancelling an operation
let operation = BlockOperation()
operation.addExecutionBlock { [unowned operation] in
  guard !operation.isCancelled else { return }
  print(Thread.current)
}

let queue = OperationQueue()
queue.addOperation(operation)
operation.cancel()
```

There's no way to set data on `OperationQueue` that gets carried by the operation.
But they support the concept of dependencies which allows you start one operation
after another one finishes. However, cancelling an operation doesn't cancel its
dependencies (similar to `Thread`).

```Swift
// Adding dependencies to an operation
let operationA = BlockOperation { print("A") }
let operationB = BlockOperation { print("B") }
operationB.addDependency(operationA)

let queue = OperationQueue()
queue.addOperation(operationA)
queue.addOperation(operationB)
```

`OperationQueue` is also smart enough to prevent thread starvation where many
threads are created that compete which each other to get CPU time. In the example
below, although 1000 operations are created, only less than 100<sup>1</sup> threads
get created (<sup>1</sup> the number depends on various factor):

```Swift
// Thread starvation
let queue = OperationQueue()
for number in 0..<1000 {
  queue.addOperation {
    print(number, Thread.current)
  }
}
```

However, operations still don't cooperate with other operations and other than
checking for cancellation, they don't give up on the thread until the operation
is finished. This means operations compete with each other even when, for instance,
we're waiting for some API call to finish and ideally want the thread to perform
some other operation in the meantime.

## Grand Central Dispatch (GCD) - iOS 8

In 2009, Apple announce GCD with iOS 8 where you no longer think of concurrences
in terms of Threads but instead in terms of queues. GCD actually empowers `OperationQueue`
under the hood but its quite simpler.

A `DispatchQueue`, is an object that manages the execution of tasks serially or
concurrently on your app's main thread or on a background thread.

Work submitted to dispatch queues executes on a pool of threads managed by the
system. Except for the dispatch queue representing your app's main thread, the
system makes no guarantees about which thread it uses to execute a task.

`DispatchQueue` is serial by default, which is a contrast to the operation
queues that are concurrent by default.

```Swift
// Sequential execution on the same thread
let queue = DispatchQueue(label: "queue")

queue.async { print("1", Thread.current) }
queue.async { print("2", Thread.current) }
queue.async { print("3", Thread.current) }
queue.async { print("4", Thread.current) }
queue.async { print("5", Thread.current) }

// 1 <NSThread: 0x600001c5ce80>{number = 2, name = (null)}
// 2 <NSThread: 0x600001c5ce80>{number = 2, name = (null)}
// 3 <NSThread: 0x600001c5ce80>{number = 2, name = (null)}
// 4 <NSThread: 0x600001c5ce80>{number = 2, name = (null)}
// 5 <NSThread: 0x600001c5ce80>{number = 2, name = (null)}
```

In the example code above, it's possible to run the operations concurrently by
providing an attribute when creating the queue like:

```Swift
let queue = DispatchQueue(label: "queue", attributes: .concurrent)
// 1 <NSThread: 0x600002a14300>{number = 2, name = (null)}
// 2 <NSThread: 0x600002a10200>{number = 3, name = (null)}
// 4 <NSThread: 0x600002a10200>{number = 3, name = (null)}
// 3 <NSThread: 0x600002a14300>{number = 2, name = (null)}
// 5 <NSThread: 0x600002a24140>{number = 4, name = (null)}
```

Just like `OperationQueue`, `DispatchQueue` can also make sure thread thread
exhaustion doesn't happen.

`DispatchQueue` is also capable of scheduling work to be performed in the future
without blocking the current thread.

```Swift
queue.asyncAfter(deadline: .now() + 1) { print(Thread.current) }
```

Similar to `OperationQueue`, it's possible to set a quality of service on the
queue. Cancelling a queue is also possible and it is done cooperatively which
means we need to check if the item of work has been cancelled to perform a
short circuit:

```Swift
let queue = DispatchQueue(label: "queue", attributes: [.concurrent])

var item: DispatchWorkItem?
item = DispatchWorkItem { [unowned item] in
  Thread.sleep(forTimeInterval: 1)
  guard item?.isCancelled == false else {
    print("It's cancelled.")
    return
  }
  print("Finished performing the item.")
}

queue.async(execute: item!)
item?.cancel()
```

Another feature of `DispatchQueue` is passing objects that can be carried on
within that execution context to be accessed later on:

```Swift
// Adding specifics
let queue = DispatchQueue(label: "queue", attributes: .concurrent)
let key = DispatchSpecificKey<String>()
queue.setSpecific(key: key, value: "My Queue")

queue.asyncAfter(deadline: .now() + 1) {
  print("Start")
  let name = DispatchQueue.getSpecific(key: key)!
  print("Name is: \(name)")
  print("Finished")
}
```

Starting up a new queue from the inside of the execution context of another
queue does not mean the specifics are automatically inherited. So, if we run
two units of work in parallel, we may lose the specifics from the root
execution context.

```Swift
// Accessing the specifics of another execution context
queue.asyncAfter(deadline: .now() + 1) {
  print("Start")
  let name = DispatchQueue.getSpecific(key: key)!
  print("Name is: \(name)")
  print("Finished")
  
  let innerQueue = DispatchQueue(label: "inner-queue", attributes: .concurrent)
  innerQueue.async {
    print("Name is: \(DispatchQueue.getSpecific(key: key) ?? "nil")")
  }
}

// Start
// Name is: My Queue
// Finished
// Name is: nil
```

There's a fix for the problem above, which is providing a target to the inner
queue. We can make even serial queues target concurrent queues; so that their
work is guaranteed to be executed sequentially but it's taking its Thread
from that shared Thread pool in the concurrent queue.

```Swift
// Inheriting the specifics of another execution context
queue.asyncAfter(deadline: .now() + 1) {
  print("Start")
  let name = DispatchQueue.getSpecific(key: key)!
  print("Name is: \(name)")
  print("Finished")
  
  let innerQueue = DispatchQueue(label: "inner-queue", attributes: .concurrent, target: queue)
  innerQueue.async {
    print("Name is: \(DispatchQueue.getSpecific(key: key) ?? "nil")")
  }
}

// Start
// Name is: My Queue
// Finished
// Name is: My Queue
```

Although there's technically a way to set data on the base queue and have it
implicitly travel through the entire execution context, it requires access
to the parent queue. In other words, queues don't inherently know what the
current execution context is. It also doesn't provide any means to cooperate
with other queues.

Although a `DispatchWorkItem` can be cancelled, there's no way to have the
cancellation of one item trickle down to child work items that are created.

In cases where we need to wait for multiple units of work that aren't
concurrently to finish before continuing, we can use `DispatchGroup`,
which allows us to treat multiple units of work as a single unit of work.

```Swift
let group = DispatchGroup()
let queueA = DispatchQueue(label: "queue", attributes: .concurrent)
let queueB = DispatchQueue(label: "queue", attributes: .concurrent)

queueA.async(group: group) {
  Thread.sleep(forTimeInterval: 0.5)
  print("Queue A finished.", Thread.current)
}
queueB.async(group: group) {
  Thread.sleep(forTimeInterval: 0.2)
  print("Queue B finished.", Thread.current)
}

group.wait()

// Queue B finished. <NSThread: 0x600003e84380>{number = 4, name = (null)}
// Queue A finished. <NSThread: 0x600003ea4100>{number = 5, name = (null)}
```

Regarding data races, GCD does provide some tools to synchronize data access.
By using `sync` function, which runs some unit of work and returns once that's
finished, and a `barrier` flag, the queue kinda uses a lock which blocks any
new work item from entering until the current one is finished.

```Swift
class Counter {
  private let queue = DispatchQueue(label: "counter", attributes: .concurrent)
  private(set) var count = 0
  
  func increment() {
    queue.sync(flags: .barrier) {
      count += 1
    }
  }
}

let counter = Counter()
for _ in 0..<1000 {
  counter.increment()
}
print(counter.count)
```

## Tasks - iOS 13

A task is the basic unit of concurrency in the system. By looking at Instrument,
can see it has the following lifetime states:

1. Creating
2. Running
3. Suspended
4. Ended

Tasks are initialized by passing a closure containing the code that will be
executed by a given task.

After this code has run to completion, the task has completed, resulting in
either a failure or result value, this closure is eagerly released.

Retaining a task object doesn't indefinitely retain the closure, because any
references that a task holds are released after the task completes. Consequently,
tasks rarely need to capture `weak` references to values.

Every asynchronous function is executing in a task. In other words, a task is
to asynchronous functions, what a thread is to synchronous functions.

So tasks seem to be capable of solving the thread explosion problem by using a
pool of threads, all without having to manage an auxiliary object like an
operation queue or dispatch queue. That’s already an improvement over threads
and queues. Therefore, running the following code won't create a thousands thread:

```Swift
for n in 0..<workCount {
  Task {
    // It won't go more than 10-15 threads (depends on the computation power)
    print(n, Thread.current)
  }
}
```

There are seven things to know when working with [task cancellation](https://www.hackingwithswift.com/quick-start/concurrency/how-to-cancel-a-task):

1. You can explicitly cancel a task by calling its `cancel()` method.
2. Any task can check `Task.isCancelled` to determine whether the task has been
cancelled or not.
3. You can call the `Task.checkCancellation()` method, which will throw a
CancellationError if the task has been cancelled or do nothing otherwise.
4. Some parts of Foundation automatically check for task cancellation and will
throw their own cancellation error even without your input.
5. If you’re using `Task.sleep()` to wait for some amount of time to pass,
cancelling your task will automatically terminate the sleep and throw a `CancellationError`.
6. If the task is part of a group and any part of the group throws an error,
the other tasks will be cancelled and awaited.
7. If you have started a task using SwiftUI’s `task()` modifier, that task will
automatically be canceled when the view disappears.

Tasks also have the ability to schedule work after waiting some time. Threads
accomplished this with a `sleep` function that was unfortunately blocking and
so would tie up thread resources, and queues accomplished this by scheduling
work to be executed on a future date. It’s not super ergonomic to have to use
nanoseconds. Although this method is called “sleep”, it is quite different from
the sleep we have used many times on Thread. This sleep does pause the current
task for an amount of time, but it does not hold up the thread. The sleep
function on Task works cooperatively so that other tasks can have a chance
to do their work on the cooperative thread pool, and then once the time
passes our task will be resumed.

We can spin up 1,000 tasks and sleep them for a second:

```Swift
for n in 0..<workCount {
  Task {
    try await Task.sleep(nanoseconds: NSEC_PER_SEC * 1000)
  }
}
```

When we await the sleep function we are able to suspend the execution of this
task, thus freeing up its thread to be used by other tasks. If we run this and
pause the executable we will see a small number of threads have been created,
but they all have just a single stack frame with a cryptic function name:

```swift
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

his `__workq_kernreturn` function is how Grand Central Dispatch parks a thread
while it waits for work to be scheduled on it. Once the suspension is over, the
work can be resumed on any thread not just the one that started off of.

### Task priority and cancellation

Like all other forms of concurrency discussed so far, tasks also support the
concept of priority, and it’s a granular value of finitely many possibilities:

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

This allows you to tell the system how important this task is, and the runtime
may be able to give this task more or less time depending on various factors.

In order to make use of cancellation we need to get a handle on the actual task
we want to cancel, which means we need to assign a variable:

```Swift
let task = Task {
  print(Thread.current)
}
```

And then at any time we can cancel it:

```Swift
task.cancel()
```

However, if we run this we will see that the print statement still executes even
though we canceled:

```Swift
<NSThread: 0x106204290>{number = 2, name = (null)}
```

This is a little different from threads, in which if you cancelled it right
after creating it we could short-circuit the thread being started. Now we don’t
know this is 100% always the case, but it seemed pretty consistent. So it seems
tasks are a bit more eager in that they start up even if we cancel right after
creating them.

The real way to short-circuit the work of a cancelled task is to manually check
if the task has been cancelled:

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

Even though `isCancelled` seems like a global value it is actually local to
just the task we are operating in. This is similar to how cancellation worked
with regular threads too.

### Current task

There is no Task.current static like there was `Thread.current`. This is
because it’s possible to not be operating inside the context of a task, whereas
you are always operating within a thread no matter what.

There is still a way to get the current task, but you have to invoke a function
with a closure and that closure is handed the current task if it exists:

```Swift
Task {
  withUnsafeCurrentTask { task in
    print(task) // Optional(Swift.UnsafeCurrentTask(_task: (Opaque Value)))
  }
}
```

If your asynchronous context happens to also be a failable context, then you
can check for cancellation via a throwing function rather than a boolean:

```Swift
let task = Task {
  try Task.checkCancellation()
  print(Thread.current)
}
task.cancel()
```

If the task has been cancelled when we try to invoke `checkCancellation`, the
function will throw, thus short-circuiting the rest of the function. This can
be a lot more ergonomic to invoke than guarding for `Task.isCancelled`.

### @TaskLocal

Tasks have a feature known as “task local values” that allow us to associate a
value with a task and then it can be effortlessly retrieved from anywhere that
runs in the context of that task.

```Swift
enum MyLocals {
  @TaskLocal static var id: Int!
}

print("before:", MyLocals.id)
MyLocals.$id.withValue(42) {
  print("withValue:", MyLocals.id!)
  Task {
    try await Task.sleep(nanoseconds: NSEC_PER_SEC)
    Task {
      print("Task:", MyLocals.id!)
    }
  }
}
print("after:", MyLocals.id)

// prints out:
// before: nil
// withValue: 42
// after: nil
// Task: 42
```

The moment we create the task it captures all of the current task locals, and
so then it doesn’t matter that later the withValue operation ends and the id
local reverts back to nil.

## Sendable Types vs Sendable Values

A type that conforms to the `Sendable` protocol is a thread-safe type: values
of that type can be shared with and used safely from multiple concurrent
contexts at once without causing data races, [_source_](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0430-transferring-parameters-and-results.md#sendable-values-and-sendable-types).

If a value does not conform to `Sendable`, Swift must ensure that the value is
never used concurrently. The value can still be sent between concurrent
contexts, but the send must be a complete transfer of the value's entire region
implying that all uses of the value (and anything non-`Sendable` typed that
can be reached from the value) must end in the source concurrency context
before any uses can begin in the destination concurrency context. Swift
achieves this property by requiring that the value is in a disconnected
region and we say that such a value is a `sending` value.

```Swift
actor MyActor {
  var myNS: NonSendable

  func g() async {
    // 'ns' is initially a `sending` value since it is in a disconnected region...
    let ns = NonSendable()

    // ... but once we assign 'ns' into 'myNS', 'ns' is no longer a sending
    // value...
    myNS = ns

    // ... causing calling 'sendToMain' to be an error.
    await sendToMain(ns)
  }
}
```

If a `sending` value's isolation region is merged into another disconnected
isolation region, then the value is still considered to be `sending` since
two disconnected regions when merged form a new disconnected region:

```Swift
func h() async {
  // This is a `sending` value.
  let ns = Nonsending()

  // This also a `sending value.
  let ns2 = NonSendable()

  // Since both ns and ns2 are disconnected, the region associated with
  // tuple is also disconnected and thus 't' is a `sending` value...
  let t = (ns, ns2)

  // ... that can be sent across a concurrency boundary safely.
  await sendToMain(ns)
}
```

Passing a non-`Sendable` instance into a method that takes a `sending` value
must be done only once, even if it's in a synchronous context. Example:

```Swift
func run() {
    let nonSendable = NonSendable()
    
    for _ in 0..<10 {
        // Sending 'nonSendable' risks causing data races
        resume(nonSendable)
    }
}

func resume(_ value: sending NonSendable) {
    print(value)
}

final class NonSendable {}
```

The same instance of `NonSendable` cannot be passed to the `resume(_:)`
multiple times given the compiler cannot reason what the callee is going
to do to the instance (is it going to mutate it, or merge it with something
else and pass it on to another method?).

## Isolation

Isolation is the mechanism that Swift uses to make data races impossible.
Isolation is specified at compile time. Everything is non-isolated by default.
You must take explicit action to change this. The isolation behavior is still
controlled by the definition. Isolation will not suddenly change unless you
decide you want to change it.

- Rule 1: isolation is controlled by definitions
- Rule 2: isolation can only change via async calls. Synchronous code cannot
change isolation.
- Rule 3: protocols can specify isolation

Whatever isolation was in effect when you create a Task will still be used
inside the Task body by default.

```Swift
@MainActor
class MyIsolatedClass {
    func myMethod() {
        Task {
          // still isolated to the MainActor here!

          // This closure doesn't have an explicit isolation
          // specification, and it's being passed to the `Task`
          // initializer, so it will be inferred to have the same
          // isolation as its enclosing context.  The enclosing
          // context is isolated to its `self` parameter, which this
          // closure captures, so this closure will also be isolated
          // that value.
        }
        
        Task.detached {
            // explicitly non-isolated, regardless
            // the enclosing scope
        }
    }
}

// no isolation for the type...
class MyClass {
    // ... and none for the function either
    func method() {
        // so this is a non-isolated context
    }
    
    func asyncMethod() async {
        // async does not affect this, so 
        // this is non-isolated too!
    }
}

class MyClass: SomeSupertype, SomeProtocol {
    // isolation here might depend on inheritance
    func method() {
    }
}

@MainActor
func doStuff() async {
    // I'm on the MainActor here!
    await anotherFunction() // have to look at the definition of anotherFunction
    // back on the main actor
}

// Represents a non-isolated function
() -> Void

// Represents a function that's isolated to that global actor
@MainActor () -> Int 

// Represents a function that's isolated to that parameter
(isolated MyActor) - > Int

actor WorldModelObject {
  var position: Point3D
  
  func linearMove(to finalPosition: Point3D, over time: Duration) {
    let originalPosition = self.position
    let motion = finalPosition - originalPosition

    // A closure can be isolated to one of its captures.
    // But Closure Isolation Control is still behind a feature flag, i.e., `ClosureIsolation`.
    gradually(over: time) { [isolated self] progressProportion in
      self.position = originalPosition + progressProportion * motion
    }
  }
}
```

Whenever you see an `await` keyword, **isolation** could change. That’s because
with other concurrency systems, runtime context is important. Definitions are all
that matter in Swift.

All this means is isolation will not suddenly change unless you decide you want
to change it. Whatever isolation was in effect when you create a Task will still
be used inside the Task body by default. `Task` uses the `@_inheritActorContext`
[underscored attribute](https://github.com/apple/swift/blob/main/docs/ReferenceGuides/UnderscoredAttributes.md)
to inherit the context.

Isolation also applies to variables:

```Swift
@MainActor
class MyIsolatedClass {
    static var value = 1 // this is also MainActor-isolated
}
```

### [Dynamic Actor Isolation (Under-Specified Protocol)](https://www.swift.org/migration/documentation/swift-6-concurrency-migration-guide/commonproblems/#Under-Specified-Protocol)

The most commonly-encountered form of this problem happens when a protocol has
no explicit isolation. In this case, as with all other declarations, this implies
non-isolated. Non-isolated protocol requirements can be called from generic code
in any isolation domain. If the requirement is synchronous, it is invalid for a
conforming type’s implementation to access actor-isolated state:

```swift
protocol Styler {
    func applyStyle()
}


@MainActor
class WindowStyler: Styler {
    func applyStyle() {
        // access main-actor-isolated state
    }
}
```

The above code produces the following error in Swift 6 mode:

```swift
 5 | @MainActor
 6 | class WindowStyler: Styler {
 7 |     func applyStyle() {
   |          |- error: main actor-isolated instance method 'applyStyle()' cannot be used to
   |                    satisfy nonisolated protocol requirement
   |          `- note: add 'nonisolated' to 'applyStyle()' to make this instance method not isolated to the actor
 8 |         // access main-actor-isolated state
 9 |     }
```

It is possible that the protocol actually should be isolated, but has not yet
been updated for concurrency. If conforming types are migrated to add correct
isolation first, mismatches will occur.

[That error has three fix-its:](https://developer.apple.com/forums/thread/760769)

1. Add `nonisolated` to `applyStyle` to make this instance method not isolated
to the actor
2. Add `@preconcurrency` to the conformance to defer isolation checking to run
time. This uses [dynamic actor isolation checking](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0423-dynamic-actor-isolation.md).
Witnesses of synchronous `nonisolated` protocol requirements when the witness is
isolated and the protocol conformance is annotated as `@preconcurrency`.
3. Mark the protocol requirement `applyStyle` `async` to allow actor-isolated
conformance.

### Types of Isolation

- None, aka non-isolated.
- Static: actor types, global actors (like `@MainActor`), and isolated parameters
- Specific actor value, i.e., They can be isolated to a specific parameter or
captured value.
- Dynamic: It can happen that the type system alone does not or cannot describe
the isolation actually used. This comes up regularly with systems built before
concurrency was a thing.

If something has isolation that you don’t want, you can opt-out with the
`nonisolated` keyword. This also can make a lot of sense for static constants
that are immutable and safe to access from other threads.

Dynamic Isolation is a good approach to gradually update the code-base.
Some forms of dynamic isolation are:

```Swift
MainActor.assumeIsolated {
    // promise the compiler this bit of code is actually already on the MainActor
}

// add a runtime check to guarantee that the actor-ness is what you expect
MainActor.preconditionIsolated()

await MainActor.run {
    // run a chunk of synchronous code, hopping over to the actor if needed
}
```

In Swift 6, [SE-0423](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0423-dynamic-actor-isolation.md)
added a new approach to replace the following dynamic isolation:

```Swift
// The protocol guarantees that respondToUIEvent runs on the main thread but isn't isolated to MainActor
public protocol ViewDelegateProtocol {
  func respondToUIEvent()
}

@MainActor
class MyViewController: ViewDelegateProtocol {
  // MyViewController.respondToUIEvent() is @MainActor-isolated, but it satisfies a nonisolated
  // protocol requirement that can be called from generic code off the main actor.
  func respondToUIEvent() { // error: @MainActor function cannot satisfy a nonisolated requirement
      // implementation...   
  }
}

// Solution: dynamic isolation: make the method nonisolated to satisfy the protocol requirement
// and wraps the implementation in MainActor.assumeIsolated
// Caveat: the programmer loses static data-race safety in their own code, because internal 
// callers of respondToUIEvent() are free to invoke it from any isolation domain without compiler errors.
@MainActor
class MyViewController: ViewDelegateProtocol {
  nonisolated func respondToUIEvent() {
    MainActor.assumeIsolated {
      // implementation...   
    }
  }
}
```

Instead of the code above, [SE-0423](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0423-dynamic-actor-isolation.md)
proposed a solution to suppress the `nonisolated` protocol conformance
requirement by using `@preconcurrency` annotation on the protocol to indicate
that the protocol itself predates concurrency:

```Swift
@MainActor
class MyViewController: @preconcurrency ViewDelegateProtocol {
 func respondToUIEvent() {
   // implementation...
 }
}
```

When importing a module that was compiled with the Swift 6 language mode into
code that is not, it's possible to call actor-isolated functions from outside
the actor using `@preconcurrency`. For example:

In the above code, `onMain` from ModuleA can be called from outside the main
actor via a call to `notIsolated()`. To close this safety hole, a dynamic check
is inserted at the call-site of `onMain()` when ModuleB is recompiled against
ModuleA after ModuleA has migrated to the Swift 6 language mode.

#### Explicit isolation

Currently, [there's a proposal](https://github.com/sophiapoirier/swift-evolution/blob/closure-isolation/proposals/nnnn-closure-isolation-control.md)
that enables closures to be fully explicit about all three types of formal isolation:

- `nonisolated`
- global actor
- specific actor value

Explicit annotation has the benefit of disabling inference rules and the
potential that they lead to a formal isolation that is not preferred.

```Swift
Task { nonisolated (parameter) in // Opting out of @inheritsIsolation
  print("nonisolated")
}

actor A {
  nonisolated func isolate() {
    Task { [isolated self] in
      print("isolated to 'self'")
    }
  }
}
```

```Swift
class NonSendableType {
  @MainActor
  func globalActor() {
    Task {
      // accessing self okay
    }
  }

  func isolatedParameter(_ actor: isolated any Actor) {
    Task {
      // not okay to access self
    }
  }
}
```

#### [isolated(any)](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0431-isolated-any-functions.md)

A function value with this type dynamically carries the isolation of the
function that was used to initialize it.

```Swift
func gradually(over: Duration, operation: @isolated(any) (Double) -> ())
```

When such a function is called from an arbitrary context, it must be assumed
to always cross an isolation boundary. This means, among other things, that
the call is effectively asynchronous and must be `await`ed.

```Swift
await operation(timePassed / overallDuration)
```

This is useful because even though it's possible to determine the isolation
through code inspection, closures, however, are special. Their isolation is
influenced not just by where there are defined, but also by what they capture.
The context of a closure definition matters, but that context is also lost when
you pass it around. Example:

```Swift
func scheduleSomeWork(_ f: @Sendable @escaping () async -> Void) {
  // We've lost all the static isolation information
  // available where f was defined.
  Task {
  await f()
  }
}

func defineClosure() {
  scheduleSomeWork { @MainActor in
    print("I'm on the main actor")
  }
}
```

There is one really important place where it makes a big difference: the
`Task` creation APIs. Which actor should that task run on? This information
is encoded in the closure body. When looking at the function’s type, it’s
completely invisible.

But `Task` has to start somewhere, so it first begins with a global executor
context. And then the very next thing that happens is the closure hops over
to the correct actor. This double hop, aside from being inefficient, has a
very significant programmer-visible effect: `Task` does not preserve order.

```Swift
Task { print("a") }
Task { print("b") }
Task { print("c") }
```

This proposal makes it possible to inspect a function value’s isolation.
A closure annotated with `@isolated(any)` can expose its captured isolation at runtime.

```Swift
func scheduleSomeWork(_ f: @Sendable @escaping @isolated(any) () async -> Void) {
  // closures have properties now!
  let isolation = f.isolation

  // do something with "isolation" I guess?
}
```

Closures can now have properties, they're just a type of course. The type
of the isolation property matches isolated parameters: `(any Actor)?`,
which is read-only. Aside from making this property available, this annotation
has a small effect on semantics. Adding `@isolated(any)` to a closure means
it must be called with an await. This is true even it does not cross an
isolation boundary. `Task` can now synchronously enqueue work onto executors.

However, I really do want to stress that “well-defined” ordering for `Task`
creation is not the same as “FIFO”. A high priority task can still execute before
more-recently-created lower-priority tasks.

It is recommended that APIs which take functions that are likely to run
concurrently and don't have a predetermined isolation take those functions as
`@isolated(any)`. This allows the API to make more intelligent scheduling
decisions about the function.

Examples that should usually use `@isolated(any)` include:

- functions that wrap the creation of a task
- algorithms that call a function multiple times in parallel, such as a parallel
`map`

Examples that should usually not use `@isolated(any)` include:

- algorithms that preserve the current isolation, such as a non-parallel `map`;
these functions should usually take a non-`Sendable` function instead
- APIs that intend to call the function with a specific isolation, such as UI
frameworks that expect their event handlers to be `@MainActor` or actor functions
that run an operation on the actor

#### Isolation inheritance

`@inheritsIsolation` unconditionally and implicitly captures the isolation context.

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

Dynamic isolation comes up regularly with systems built before concurrency was
a thing. One tool we have to deal with this is dynamic isolation. These are APIs
that allow us to express isolation in a way that is invisible by just looking
at definitions.

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

Isolation cannot change at all for a synchronous function. And it cannot change
for the synchronous parts of an async function. But, as soon as you need to
make an async call, it could.

This is really a consequence of the fact that isolation is controlled entirely
by a function’s definition. It does not matter how the caller is being isolated.
This is completely different from how queues or locks work.

#### `sending` parameter and result values

This [proposal](https://github.com/swiftlang/swift-evolution/blob/main/proposals/0430-transferring-parameters-and-results.md)
extends region isolation to enable the application of an explicit `sending`
annotation to function parameters and results. A function parameter or result
that is annotated with `sending` is required to be disconnected at the function
boundary and thus possesses the capability of being safely sent across an
isolation domain or merged into an actor-isolated region in the function's
body or the function's caller respectively.

### Passing non-sendable types into actor-isolated context

```Swift
actor Foo {
    init(name: NonSendable) {
        //
    }
    
    init(name: @Sendable () -> NonSendable) {
        //
    }
}

class NonSendable {
    var name: String = "Name"
}

// Not allowed: Passing argument of non-sendable type 'NonSendable' into actor-isolated context 
_ = Foo(name: NonSendable())

// Allowed. Could even use autoclosure or create a boxing type.
_ = Foo(name: { NonSendable() })

```

It is also possible to make the whole function be `Sendable`

```Swift
// @Sendable must be placed before the ACL
@Sendable private func foo() {
    //
}
```

## Concurrency-safe singletons

There's three options:

- Make the type conform to `Sendable`. Ideally, this needs to be placed in
the same file that the class was declared. Otherwise, need to use
`@unchecked Sendable` for retroactive conformance.
- Isolate the class by a global actor such as `@MainActor`, making its
access serialized and concurrency-safe. However, actor isolation might not
always work since it complicates access from non-concurrency contexts.
- Using `nonisolated(unsafe)` keyword

In Swift, global variables are initialized lazily on the first access. In
Swift 6, this is done atomically to make sure there is no data races when
the different threads try to access and initialize the variable at the same time.

## Actor

From _Visualize and optimize Swift concurrency_, WWDC22:
> Actors make it safe for multiple tasks to manipulate shared state. However,
> they do this by serializing access to that shared state. Only one task at a
> time is allowed to occupy the Actor, and other tasks that need to use that
> Actor will wait. Swift concurrency allows for parallel computation using
> unstructured tasks, task groups, and async let. Ideally, these constructs
> are able to use many CPU cores simultaneously. When using Actors from such
> code, beware of performing large amounts of work on an Actor that's shared
> among these tasks. When multiple tasks attempt to use the same Actor
> simultaneously, the Actor serializes execution of those tasks. Because of
> this, we lose the performance benefits of parallel computation.
>
> This is because each task must wait for the Actor to become available. To fix
> this, we need make sure that tasks only run on the Actor when they really need
> exclusive access to the Actor's data. Everything else should run off of the
> Actor. We divide the task into chunks. Some chunks must run on the Actor,
> and the others don't. The non-Actor isolated chunks can be executed in
> parallel, which means the computer can finish the work much faster.

While actors are great for protecting encapsulated state, sometimes we want to
modify and read individual properties on the type, so actors aren't quite the
right tool for this. Furthermore, we can't guarantee the order that operations
run on an actor, so we can't ensure that, for instance, our cancellation will
run first. We'll need something else such as atomic types.

When a method on actor is marked as `nonisolated`, we tell the Swift compiler
that we don't need access to the shared state of the actor. But if the body
of that function actually needs access to a shared state, then the `nonisolated`
method could be marked as `async` and then in the body, mark all the actor
states with the `await` keyword.

```Swift
actor Counter {
  private var count = 0

  nonisolated func increment() async -> Int {
    await count += 1
    return await count
  }
}
```

Actor-isolated functions are reentrant. When an actor-isolated function
suspends, reentrancy allows other work to execute on the actor before
the original actor-isolated function resumes, which we refer to as
interleaving. Reentrancy eliminates a source of deadlocks, where two
actors depend on each other, can improve overall performance by not
unnecessarily blocking work on actors, and offers opportunities for
better scheduling of (e.g.) higher-priority tasks. However, it means
that actor-isolated state can change across an `await` when an interleaved
task mutates that state, meaning that developers must be sure not to
break invariants across an await. In general, this is the reason for
requiring `await` on asynchronous calls, because various state (e.g.,
global state) can change when a call suspends.

## @MainActor

`@MainActor` is a global actor that uses the main queue for executing its work.
In practice, this means methods or types marked with `@MainActor` can (for the
most part) safely modify the UI because it will always be running on the main
queue, and calling `MainActor.run()` will push some custom work of your choosing
to the main actor, and thus to the main queue.

Whenever you use `@StateObject` or `@ObservedObject` inside a view, Swift will
ensure that the whole view runs on the main actor so that you can’t accidentally
try to publish UI updates in a dangerous way.

The `body` property of your SwiftUI views is always run on the main actor.

If you need certain methods or computed properties to opt out of running on the
main actor, use `nonisolated` as you would do with a regular actor

More broadly, any type that has `@MainActor` objects as properties will also
implicitly be `@MainActor` using _global actor inference_ – a set of rules
that Swift applies to make sure global-actor-ness works without getting in
the way too much.

If you do need to spontaneously run some code on the main actor, you can do
that by calling `MainActor.run()` and providing your work. This allows you
to safely push work onto the main actor no matter where your code is currently
running, like this:

```Swift
func couldBeAnywhere() async {
    await MainActor.run {
        print("This is on the main actor.")
    }
}

await couldBeAnywhere()
```

You can send back nothing from `run()` if you want, or send back a value like this:

```Swift
func couldBeAnywhere() async {
    let result = await MainActor.run { () -> Int in
        print("This is on the main actor.")
        return 42
    }

    print(result)
}

await couldBeAnywhere()
```

Even better, if that code was already running on the main actor then the code
is executed immediately – it won’t wait until the next run loop in the same
way that `DispatchQueue.main.async()` would have done.

If you wanted the work to be sent off to the main actor without waiting for
its result to come back, you can place it in a new task like this:

```Swift
func couldBeAnywhere() {
    Task {
        await MainActor.run {
            print("This is on the main actor.")
        }
    }

    // more work you want to do
}

couldBeAnywhere()
```

Or you can also mark your task’s closure as being @MainActor, like this:

```Swift
func couldBeAnywhere() {
    Task { @MainActor in
        print("This is on the main actor.")
    }

    // more work you want to do
}

couldBeAnywhere()
```

This is particularly helpful when you’re inside a synchronous context, so
you need to push work to the main actor without using the `await` keyword.

**Important:** If your function is already running on the main actor, using
`await MainActor.run()` will run your code immediately without waiting for
the next run loop, but using Task as shown above will wait for the next run loop.

You can see this in action in the following snippet:

```Swift
@MainActor class ViewModel: ObservableObject {
    func runTest() async {
        print("1")

        await MainActor.run {
            print("2")

            Task { @MainActor in
                print("3")
            }

            print("4")
        }

        print("5")
    }
}

// prints out:
// 1 2 4 5 3
```

That marks the whole type as using the main actor, so the call to
`MainActor.run()` will run immediately when runTest() is called. However,
the inner Task will not run immediately, so the code will print `1, 2, 4, 5, 3`.

You can mark individual methods with the attribute as well:

```Swift
@MainActor func updateViews() {
    // Perform UI updates..
}
```

And you can even mark closures to perform on the main thread:

```Swift
func updateData(completion: @MainActor @escaping () -> ()) {
    Task {
        await someHeavyBackgroundOperation()
        await completion()
    }
}
```

The `MainActor` in Swift comes with an extension to use the actor directly:

```Swift
extension MainActor {
    /// Execute the given body closure on the main actor.
    public static func run<T>(
      resultType: T.Type = T.self,
      body: @MainActor @Sendable () throws -> T
    ) async rethrows -> T
}

Task {
    await someHeavyBackgroundOperation()

    // MainActor.run {} has a problem: just like all forms of runtime synchronization,
    // the compiler cannot help you. In most cases it's better to annotate the whole
    // type with @MainActor
    await MainActor.run {
        // Perform UI updates
    }
}
```

The `run` method is useful when we need to execute multiple synchronous calls
without the risk of suspension:

```Swift
// atomic
await MainActor.run {
    obj.methodA()
    obj.methodB()
}

// definitely not atomic
await obj.methodA()
await obj.methodB()   // There is a suspension point between A and B

// equivalent to .run
await Task { @MainActor in
    m.methodA()
    m.methodB()
}.value
```

## Testing

There are no tools that allow us to deterministically assert on what happens in
between units of async work. We have to sprinkle in some `Task.yields` and hope
it’s enough, and as we’ve seen a few times now, often it is not enough. We should
probably be yielding a lot more times in these tests, and possibly even waiting
for a duration of time to pass, which would unfortunately slow down our test
suite. And still we could never be 100% certain that the test still won’t flake
some day.

For instance, imagine tapping a button starts a task which does an async call.
Then tapping another button is supposed to cancel the running task. The test
can fail sometimes because for whatever reason the Swift concurrency runtime
is not scheduling the task quickly enough for us to cancel it.

```Swift
let task = Task { await model.getFactButtonTapped() }
await Task.yield()
await Task.yield()
await Task.yield()
await Task.yield()
model.cancelButtonTapped()
```

One way to do this is creating a mega yield function, or since we have access
to the task property, call `await task.value` (this isn't always possible given
sometimes the task is private). Looking at how Apple does it in the open-sourced
repo [swift-async-algorithms](https://github.com/apple/swift-async-algorithms),
we can see they [override a global variable](https://github.com/apple/swift-async-algorithms/blob/adb12bfcccaa040778c905c5a50da9d9367fd0db/Sources/AsyncSequenceValidation/Test.swift#L319):

```Swift
swift_task_enqueueGlobal_hook = { job, original in
  Context.driver?.enqueue(job)
}
```

Looking at the symbol, `swift_task_enqueueGlobal_hook`, can find an equivalent
in the `C` header file [_CAsyncSequenceValidationSupport.h](https://github.com/apple/swift-async-algorithms/blob/adb12bfcccaa040778c905c5a50da9d9367fd0db/Sources/_CAsyncSequenceValidationSupport/_CAsyncSequenceValidationSupport.h#L242-L247).
There's also another signature in the [Concurrency.h](https://github.com/apple/swift/blob/bd11fceff5390c960ccdfca2f6c8951dc96941c9/include/swift/Runtime/Concurrency.h#L737-L738)
file in the Concurrency repo. The Async Algorithms package wants access to some `C`
functions that are defined in Swift’s C++ codebase, [GlobalExecutor.cpp](https://github.com/apple/swift/blob/bd11fceff5390c960ccdfca2f6c8951dc96941c9/stdlib/public/BackDeployConcurrency/GlobalExecutor.cpp#L85-L92),
but are not exposed with a proper Swift API. In order to do that the package
defines a system library that publicly declares those signatures, and that makes
it available in Swift. Whenever an async task is enqueued it calls this closure
for processing. And in this case “task” does not refer to a literal unstructured
task created with the `Task` type. Here “task” means any kind of asynchronous work,
including every single suspension point from an await.

The way to dynamically open libraries in Swift is to start with `dlopen`,
a `C` function, which means we can even see its documentation from the man pages
in terminal:

```Bash
$ man dlopen

NAME
dlopen -- load and link a dynamic library or bundle

SYNOPSIS
#include <dlfcn.h>

void*
dlopen(const char* path, int mode);
```

In order to do a similar thing, we'll need to create a glue code to override
the `C` variable:

```Swift
import Darwin

typealias Original = @convention(thin) (UnownedJob) -> Void
typealias Hook = @convention(thin) (UnownedJob, Original) -> Void

var swift_task_enqueueGlobal_hook: Hook? {
  get { _swift_task_enqueueGlobal_hook.pointee }
  set { _swift_task_enqueueGlobal_hook.pointee = newValue }
}

private let _swift_task_enqueueGlobal_hook =
  // The man pages suggest using RTLD_LAZY as a default
  
  // dlopen: to dynamically open a library. It returns `UnsafeMutableRawPointer?`, 
  // which can be used to look up the address of a particular symbol using another C function, dlsym

  // dlsym: to lookup address of a particular C function
  dlsym(dlopen(nil, RTLD_LAZY), "swift_task_enqueueGlobal_hook")
    .assumingMemoryBound(to: Hook?.self)
```

Usage:

```Swift
swift_task_enqueueGlobal_hook = { job, original in 
  print("Job enqueued", job)
  original(job)
}
```
